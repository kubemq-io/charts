# KubeMQ Charts
KubeMQ is a Cloud Native, enterprise grade message queue broker for distributed services architecture.

KubeMQ is delivered as a small, lightweight Docker container, designed for any type of workload and architecture running in Kubernetes or any other containers orchestration system which support Docker.

## Requirements
KubeMQ Helm charts require **Helm v3.8+ or Helm v4**. Please download/upgrade from [https://github.com/helm/helm](https://github.com/helm/helm).

> **Prerelease channel.** The v3 generation of the charts is published on the `-next` prerelease
> channel, so every command below includes `--devel`. Without `--devel`, Helm skips prerelease
> versions and the install will not resolve a v3 chart.

## Add KubeMQ Helm Repository

```
$ helm repo add kubemq-charts https://kubemq-io.github.io/charts
$ helm repo update
```

## Install KubeMQ (three charts)

Install the CRDs, then the controller (operator), then a cluster:

``` console
$ helm install --devel --create-namespace -n kubemq kubemq-crds kubemq-charts/kubemq-crds
$ helm install --devel --wait -n kubemq kubemq-controller kubemq-charts/kubemq-controller
$ helm install --devel --wait -n kubemq kubemq-cluster --set key={your-license-key} kubemq-charts/kubemq-cluster
```

## Install KubeMQ (umbrella — one release)

The `kubemq` umbrella chart bundles the CRDs, the operator, and a single `KubemqCluster` in a
single release:

``` console
$ helm install --devel --create-namespace --wait -n kubemq kubemq kubemq-charts/kubemq --set key={your-license-key}
```

## Using Private Container Registries

Both KubeMQ Controller and KubeMQ Cluster charts support pulling images from private container registries. To use private registries, you need to:

1. **Create a registry secret:**

``` console
$ kubectl create secret docker-registry my-registry-secret \
  --docker-server=my-private-registry.com \
  --docker-username=myuser \
  --docker-password=mypassword \
  --docker-email=myemail@example.com \
  --namespace=kubemq
```

2. **Install the controller with the registry secret:**

``` console
$ helm install --devel --wait -n kubemq kubemq-controller kubemq-charts/kubemq-controller \
  --set imagePullSecrets[0].name=my-registry-secret
```

**For kubemq-cluster:**

``` console
$ helm install --devel --wait -n kubemq kubemq-cluster kubemq-charts/kubemq-cluster \
  --set key={your-license-key} \
  --set imagePullSecrets[0].name=my-registry-secret
```

3. **Multiple registry secrets (if needed):**

``` console
$ helm install --devel --wait -n kubemq kubemq-controller kubemq-charts/kubemq-controller \
  --set imagePullSecrets[0].name=registry-secret-1 \
  --set imagePullSecrets[1].name=registry-secret-2
```

4. **Using values file:**

Create a `values.yaml` file:
```yaml
imagePullSecrets:
  - name: my-registry-secret
  - name: another-registry-secret
```

Then install:
``` console
$ helm install --devel --wait -n kubemq kubemq-controller kubemq-charts/kubemq-controller -f values.yaml
```

## Uninstall KubeMQ Cluster Chart

To uninstall/delete the kubemq-release deployment:

``` console
$ helm uninstall -n kubemq kubemq-cluster
$ helm uninstall -n kubemq kubemq-controller
$ helm uninstall -n kubemq kubemq-crds
```

## Configuration

Each chart is a thin passthrough over its Kubernetes resource / values — any documented field can
be set with `--set` or a `-f values.yaml`. For the full, current set of options and starter
recipes, see each chart's own docs (these are kept in sync with the code; the tables previously
inlined here drifted out of date and were removed in favor of these sources):

- **kubemq-cluster:** [values_example.yaml](kubemq-cluster/values_example.yaml) · [README](kubemq-cluster/README.md)
- **kubemq (umbrella):** [values_example.yaml](kubemq/values_example.yaml) · [README](kubemq/README.md)
- **kubemq-controller:** [values_example.yaml](kubemq-controller/values_example.yaml) · [README](kubemq-controller/README.md)
- **kubemq-crds:** [README](kubemq-crds/README.md)

## Documentation
Please visit [https://docs.kubemq.io](https://docs.kubemq.io) for more information about KubeMQ.
