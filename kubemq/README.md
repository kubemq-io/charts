# kubemq

`kubemq` is the **umbrella** Helm chart that installs the whole KubeMQ stack in a single
release: the CRDs (bundled under `crds/`), the `kubemq-operator` Deployment, and one
`KubemqCluster` custom resource. Use it for a batteries-included install; use the
`kubemq-crds` + `kubemq-controller` + `kubemq-cluster` charts separately when you want to
manage those pieces independently.

## Installing

For example:
```console
$ helm repo add kubemq-charts  https://kubemq-io.github.io/charts
$ helm install --create-namespace -n kubemq kubemq kubemq-charts/kubemq \
  --set key={your-license-key}
```

A license `key` is **required** — the render fails without it.

## Configuration

The chart is a thin passthrough over the `KubemqCluster` custom resource. Every value you
set (except `key` and `imagePullSecrets`, which are handled specially) is rendered verbatim
into `spec:` of the generated `KubemqCluster` — so **any `KubemqCluster.spec.*` field is a
Helm value of the same name**.

[`values_example.yaml`](values_example.yaml) is a small set of **copy-and-adapt starter
recipes** (plain cluster, the connector one-liner, external Kafka, `expose`, the
`env`/`envFromSecrets` overlay, and the Docker single-node recipes) — not an exhaustive
list. The **full field catalog** (every field, its default, and the matching env var) is
[`docs/10-configuration-reference.md`](https://github.com/kubemq-io/kubemq-server/blob/master/docs/10-configuration-reference.md)
in the KubeMQ server repo, and the validating schema is the
[`KubemqCluster` CRD](crds/kubemqclusters.core.k8s.kubemq.io.crd.yaml).

> **Passthrough pinning caveat:** because every value is rendered straight into `spec:`,
> putting an *active* (uncommented) value in your `values.yaml` **pins** that field on the
> CR. The server default for that field then no longer applies — the chart value wins, even
> across upgrades. Keep your `values.yaml` minimal (license + only the fields you truly need
> to override) and leave everything else commented so the server defaults stay in effect.
> Treat `values_example.yaml` as **starter recipes** to copy-and-adapt one at a time, and
> [`docs/10-configuration-reference.md`](https://github.com/kubemq-io/kubemq-server/blob/master/docs/10-configuration-reference.md)
> as the full catalog of what *can* be set.

### Zero-config Kafka

On a **fresh** cluster, `kafka.enabled: true` is the whole story:

```yaml
key: your-license-key
kafka:
  enabled: true
```

The operator auto-selects the `next` storage engine, records it in the
`core.k8s.kubemq.io/established-engine` annotation, best-effort writes `store.engine: next`
back into the spec, and opens an in-cluster **plaintext** endpoint at
`<cluster>-kafka.<namespace>.svc:9092`. In-cluster clients need no networking config.
Producers should use `acks>=1` (the default install runs `replicas: 3`; `acks=0` can be
silently dropped on a follower). For clients **outside** the cluster, set `kafka.expose`
(NodePort/LoadBalancer) plus `kafka.advertisedHost` and `kafka.advertisedPort` to the
externally reachable name/port.

### LEAN-CRD escape hatch (`spec.env` / `spec.envFromSecrets`)

The CRD types only the **day-1** surface of each connector (enable, port(s), advertised
endpoint, `expose`, credentials ref). Every other current or future server tunable is
reachable from the CR **without a CRD/operator/chart upgrade** through two overlay fields:

- **`spec.env`** — a `map[string]string` of server env keys → values, applied **last-wins**
  over all typed emits (keys uppercased, empty values dropped, changing it rolls the pods).
  Operator-owned identity keys are **rejected** with a `ReconcileError` (no env change,
  pods untouched): `STORE_ENGINE`, `CLUSTER_ENABLE`, `CLUSTER_NAME`, `CLUSTER_ROUTES`,
  `API_BIND_ADDRESS`, `CHECKSUM`, `POD_NAME`, and any `CLUSTER_REPLICATION_*` key. Use the
  typed fields for those. **`spec.env` is not for secrets.**

  ```yaml
  env:
    CONNECTORS_KAFKA_FETCH_MAX_BYTES: "10485760"
  ```

- **`spec.envFromSecrets`** — a list of existing Secret names whose keys are injected as
  container env (`envFrom`). Secret **values never transit the operator**; standard
  Kubernetes `envFrom` precedence applies; secret-**content** changes do **not** roll the
  pods. This is the credentials path.

  ```yaml
  envFromSecrets:
    - my-kafka-credentials
  ```

## Upgrading

### Upgrading CRDs (read this first)

**The PRIMARY way to upgrade a CRD is `kubectl apply -f <canonical>`** — Helm **never
upgrades CRDs** that ship in a chart's `crds/` directory (they are installed once on first
install and left untouched on every subsequent `helm upgrade`). This applies to this
umbrella chart's bundled `crds/` too. To move the CRD schema forward, apply the canonical
CRD directly:

```console
$ kubectl apply -f https://raw.githubusercontent.com/.../kubemqclusters.core.k8s.kubemq.io.crd.yaml
```

Adopting an already-installed CRD into a chart requires Helm adoption annotations; without
them, chart adoption fails. When a release note (below) says "upgrade CRDs", it means run
the `kubectl apply` above — not `helm upgrade`.

### Zero-config coherence — engine auto-select, `spec.env`/`expose`, write-back

This release makes `kafka.enabled: true` a one-flag story (auto-selected `next` engine on a
fresh cluster, defaulted in-cluster advertised host), adds the `spec.env` /
`spec.envFromSecrets` overlay and per-connector `expose`, and records the established engine
in a CR annotation with best-effort spec write-back.

**Upgrade order: CRDs + operator as ONE step → server images → charts.** The
CRD-upgraded / operator-old window is transient, not a resting state. Read these release
notes before upgrading:

1. **Old operator strips new fields.** Do not add `spec.env`/`spec.envFromSecrets`/`expose`/
   `nodePort` (or rely on engine write-back) until the operator is upgraded — the old
   operator permanently strips unknown new fields from the CR on its first reconcile
   (full-object update on the finalizer path); re-apply the fields after the operator
   upgrade to restore them.

2. **Operator rollback is not engine-safe.** Pin `spec.store.engine: next` explicitly
   BEFORE rolling back the operator; clustered next CRs crashloop under the old operator
   (peers elided). The `core.k8s.kubemq.io/established-engine` annotation survives the old
   operator (untyped metadata), so re-upgrade re-derives the engine correctly.

3. **Explicit-legacy CRs get one checksum roll.** A CR with `spec.store.engine: legacy`
   explicitly set now emits `STORE_ENGINE=legacy` (previously elided). This changes the
   ConfigMap checksum once, causing a single rolling restart on the first reconcile after
   upgrade. Auto-selected and unset CRs are unaffected.

4. **Write-back arms the replicas freeze.** A successful engine write-back inserts
   `engine: next` into formerly-unset CRs, which retroactively arms the existing CEL
   replicas-freeze rule — replica changes on kafka / auto-next clusters now reject at
   admission. Scale next-engine clusters per the next-engine scaling docs, not by editing
   `replicas`.

Please also refer to the release notes of each version of the helm charts.
These can be found [here](https://github.com/kubemq-io/charts/releases).

## Uninstalling

```console
$ helm uninstall -n kubemq kubemq
```

The command removes all the Kubernetes components associated with the chart. If you want to
keep the history use the `--keep-history` flag. Note that CRDs installed from the `crds/`
directory are **not** removed by `helm uninstall` — delete them manually if required.
