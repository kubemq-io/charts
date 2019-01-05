# KubeMQ Charts
KubeMQ is a Cloud Native, enterprise grade message broker for distributed services architecture.

KubeMQ is delivered as a small, lightweight Docker container, designed for any type of workload and architecture running in Kubernetes or any other containers orchestration system which support Docker.


## Add KubeMQ Helm Repository

``` 
$ helm repo add kubemq-charts  https://kubemq-io.github.io/charts
```

Verify kubemq helm repository charts is properly configured by:

``` 
$ helm repo list
```

## Install KubeMQ Chart
To install KubeMQ chart with the release name kubemq-release:

``` 
$ helm install --name kubemq-release --set token={your kubemq token} kubemq-charts/kubemq 
```

## Uninstall KubeMQ Chart

To uninstall/delete the kubemq-release deployment:

``` 
$ helm delete kubemq-release
```

## Configuration

The following table lists the configurable parameters of the KubeMQ chart and their default values.


| Parameter                           | Default           | Description                                                                                 |
|:-----------------------------------|:------------------|:--------------------------------------------------------------------------------------------|
| nameOverride                       | `kubemq-cluster`  | Sets deployment name                                                                        |
| token                              | ``                | Sets KubeMQ token                                                                           |
| replicaCount                       | `3`               | Number of KubeMQ nodes                                                                      |
| cluster.enable                     | `true`            | Enable/Disable cluster mode                                                                 |
| image.repository                   | `kubemq/kubemq`   | KubeMQ image name                                                                           |
| image.tag                          | `latest`          | KubeMQ image tag                                                                            |
| image.pullPolicy                   | `Always`          | Image pull policy                                                                           |
| service.type                       | `ClusterIP`       | Sets KubeMQ service type                                                                    |
| service.apiPort                    | `8080`            | Sets KubeMQ service Api Port                                                                |
| service.restPort                   | `9090`            | Sets KubeMQ service Rest Port                                                               |
| service.grpcPort                   | `5000`            | Sets KubeMQ service gRPC Port                                                               |
| service.clusterPort                | `5228`            | Sets KubeMQ service Cluster Port                                                            |
| livenessProbe.enabled              | `true`            | Enable/Disable liveness prob                                                                |
| livenessProbe.initialDelaySeconds  | `4`               | Delay before liveness probe is initiated                                                    |
| livenessProbe.periodSeconds        | `10`              | How often to perform the probe                                                              |
| livenessProbe.timeoutSeconds       | `5`               | When the probe times out                                                                    |
| livenessProbe.failureThreshold     | `6`               | Minimum consecutive successes for the probe to be considered successful after having failed |
| livenessProbe.successThreshold     | `1`               | Minimum consecutive failures for the probe to be considered failed after having succeeded   |
| readinessProbe.enabled             | `true`            | Enable/Disable readiness prob                                                               |
| readinessProbe.initialDelaySeconds | `1`               | Delay before readiness probe is initiated                                                   |
| readinessProbe.periodSeconds       | `10`              | How often to perform the probe                                                              |
| readinessProbe.timeoutSeconds      | `5`               | When the probe times out                                                                    |
| readinessProbe.failureThreshold    | `6`               | Minimum consecutive failures for the probe to be considered failed after having succeeded   |
| readinessProbe.successThreshold    | `1`               | Minimum consecutive successes for the probe to be considered successful after having failed |
| statefulset.updateStrategy         | `RollingUpdate`   | Statefulsets Update strategy                                                                |
| volume.enabled                     | `false`           | Enable/Disable Persistence Volume Claim template                                            |
| volume.size                        | `1Gi`             | Set volume size                                                                             |
| volume.mountPath                   | ` "/store" `      | Sets container mounting point                                                               |
| volume.accessMode                  | `"ReadWriteOnce"` | Sets Persistence access mode                                                                |

Specify each parameter using the `--set key=value[,key=value]` argument to helm install. For example,
```
helm install --name kubemq-release --set token={your kubemq token},nameOverride=my-kubemq-cluster kubemq-charts/kubemq 
```

Will install KubeMQ cluster with application name set to `my-kubemq-cluster` .








