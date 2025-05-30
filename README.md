# KubeMQ Charts
KubeMQ is a Cloud Native, enterprise grade message queue broker for distributed services architecture.

KubeMQ is delivered as a small, lightweight Docker container, designed for any type of workload and architecture running in Kubernetes or any other containers orchestration system which support Docker.

## HELM
KubeMQ Helm charts required Helm v3. Please download/upgrade from [https://github.com/helm/helm](https://github.com/helm/helm)

## Add KubeMQ Helm Repository

``` 
$ helm repo add kubemq-charts  https://kubemq-io.github.io/charts
```

Verify KubeMQ helm repository charts is properly configured by:

## Update KubeMQ Helm Repository
``` 
$ helm repo update
```

## Install KubeMQ Cluster Chart

``` console 
$ helm install --create-namespace -n kubemq kubemq-crds kubemq-charts/kubemq-crds
$ helm install --wait -n kubemq kubemq-controller kubemq-charts/kubemq-controller
$ helm install --wait -n kubemq kubemq-cluster --set key={your-license-key} kubemq-charts/kubemq-cluster
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
$ helm install --wait -n kubemq kubemq-controller kubemq-charts/kubemq-controller \
  --set imagePullSecrets[0].name=my-registry-secret
```

**For kubemq-cluster:**

``` console
$ helm install --wait -n kubemq kubemq-cluster kubemq-charts/kubemq-cluster \
  --set key={your-license-key} \
  --set imagePullSecrets[0].name=my-registry-secret
```

3. **Multiple registry secrets (if needed):**

``` console
$ helm install --wait -n kubemq kubemq-controller kubemq-charts/kubemq-controller \
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
$ helm install --wait -n kubemq kubemq-controller kubemq-charts/kubemq-controller -f values.yaml
```

## Uninstall KubeMQ Cluster Chart

To uninstall/delete the kubemq-release deployment:

``` console
$ helm uninstall -n kubemq kubemq-cluster
$ helm uninstall -n kubemq kubemq-controller
$ helm uninstall -n kubemq kubemq-crds
```

## Configuration

The following table lists the configurable parameters of the KubeMQ chart and their default values.

### KubeMQ Cluster Configuration

```yaml
# Number of replicas of KubeMQ Nodes - https://docs.kubemq.io/configuration/cluster/default-template
replicas: 3

# KubeMQ license key
key: kubemq license key

# KubeMQ license data - https://docs.kubemq.io/configuration/cluster/set-license
license: kubemq license data

# Private Registry Configuration - Reference existing secrets for pulling images from private registries
imagePullSecrets: []
# Example:
#   imagePullSecrets:
#     - name: my-registry-secret
#     - name: another-registry-secret

# KubeMQ Volume Configuration - https://docs.kubemq.io/configuration/cluster/set-persistence-volume
volume:
  size: 10Gi
  storageClass: default

# KubeMQ docker image - https://docs.kubemq.io/configuration/cluster/set-cluster-image
image:
  image: kubemq/kubemq:latest
  pullPolicy: Always


# KubeMQ Api interface - https://docs.kubemq.io/configuration/cluster/set-api-interface
api:
  disabled: false
  expose: NodePort
  nodePort: 32080
  port: 8080

# KubeMQ gRPC interface - https://docs.kubemq.io/configuration/cluster/set-grpc-interface
grpc:
  disabled: false
  expose: NodePort
  nodePort: 32000
  port: 50000
  bodyLimit: 10000000
# KubeMQ REST interface - https://docs.kubemq.io/configuration/cluster/set-rest-interface
rest:
  bodyLimit: 1000000
  disabled: true
  expose: NodePort
  nodePort: 32090
  port: 9090

# KubeMQ Authentication Configuration - https://docs.kubemq.io/configuration/cluster/set-authentication
authentication:
  key: jwt
  type: jwt token type

# KubeMQ Authorization Configuration - https://docs.kubemq.io/configuration/cluster/set-authorization
authorization:
  autoReload: 300000
  policy: policy type
  url: remote url

# KubeMQ Health Configuration - https://docs.kubemq.io/configuration/cluster/set-health-probe
health:
  failureThreshold: 3
  initialDelaySeconds: 3
  periodSeconds: 4
  successThreshold: 1
  timeoutSeconds: 10

# KubeMQ Logging Configuration - https://docs.kubemq.io/configuration/cluster/set-logs
log:
  file: path to log file
  level: 1

# KubeMQ NodeSelectors Configuration - https://docs.kubemq.io/configuration/cluster/set-node-selectors
nodeSelectors:
  keys:
    key: value

# KubeMQ Queue Configuration - https://docs.kubemq.io/configuration/cluster/set-queues-settings
queue:
  defaultVisibilitySeconds: 0
  defaultWaitTimeoutSeconds: 0
  maxDelaySeconds: 0
  maxExpirationSeconds: 0
  maxReQueues: 0
  maxReceiveMessagesRequest: 0
  maxVisibilitySeconds: 0
  maxWaitTimeoutSeconds: 0

# KubeMQ Resources Configuration - https://docs.kubemq.io/configuration/cluster/set-resources-limits
resources:
  limitsCpu: "3"
  limitsEphemeralStorage: 100Gi
  limitsMemory: 2Gi
  requestsCpu: "3"
  requestsEphemeralStorage: 200Gi
  requestsMemory: 4Gi

# KubeMQ Routing Configuration - https://docs.kubemq.io/configuration/cluster/set-routing
routing:
  autoReload: 300000
  data: routing data
  url: routing url

# KubeMQ Cluster Configuration - when standalone is true, KubeMQ will run as a single node
standalone: false

# KubeMQ Store Configuration - https://docs.kubemq.io/configuration/cluster/set-store-settings
store:
  clean: true
  maxChannelSize: 0
  maxChannels: 0
  maxMessages: 0
  maxSubscribers: 0
  messagesRetentionMinutes: 0
  path: path to store
  purgeInactiveMinutes: 0

# KubeMQ TLS Configuration - https://docs.kubemq.io/configuration/cluster/set-tls
tls:
  ca: ca data
  cert: cert data
  key: key data
```

### KubeMQ Controller Configuration

```yaml
# KubeMQ Controller Image Configuration
operatorImage: docker.io/kubemq/kubemq-operator:latest
kubemqImage: docker.io/kubemq/kubemq:latest
connectorTargetsImage: kubemq/kubemq-targets:latest
connectorSourcesImage: kubemq/kubemq-sources:latest
connectorBridgesImage: kubemq/kubemq-bridges:latest

# Private Registry Configuration - Reference existing secrets for pulling images from private registries
imagePullSecrets: []
# Example:
#   imagePullSecrets:
#     - name: my-registry-secret
#     - name: another-registry-secret
```

## Documentation
Please visit [https://docs.kubemq.io](https://docs.kubemq.io) for more information about KubeMQ.




