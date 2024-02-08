# OpenTelemetry Demo Helm Chart

The helm chart installs [OpenTelemetry Demo](https://github.com/open-telemetry/opentelemetry-demo)
in kubernetes cluster.

## Prerequisites

- Kubernetes 1.24+
- Helm 3.9+

## Installing the Chart

Add OpenTelemetry Helm repository:

Pull this repository then run the following. Replace \<namespace> and \<release>. I used rvander-oteldemo for both to make it easy to find.
```console
helm package opentelemetry-demo-cco
helm install -n<namespace> <release> opentelemetry-demo-cco-0.27.3.tgz
```

## Upgrading

See [UPGRADING.md](UPGRADING.md).

## OpenShift

Installing the chart on OpenShift requires the following additional steps:

1. Create a new project:

```console
oc new-project opentelemetry-demo
```

2. Create a new service account:

```console
oc create sa opentelemetry-demo
```

3. Add the service account to the `anyuid` SCC (may require cluster admin):

```console
oc adm policy add-scc-to-user anyuid -z opentelemetry-demo
```

4. Install the chart with the following command:

```console
helm install my-otel-demo charts/opentelemetry-demo \
    --namespace opentelemetry-demo \
    --set serviceAccount.create=false \
    --set serviceAccount.name=opentelemetry-demo \
    --set grafana.rbac.create=false \
    --set grafana.serviceAccount.create=false \
    --set grafana.serviceAccount.name=opentelemetry-demo
```

## Chart Parameters

Chart parameters are separated in 4 general sections:
* Default - Used to specify defaults applied to all demo components
* Components - Used to configure the individual components (microservices) for
the demo
* Observability - Used to enable/disable dependencies
* Sub-charts - Configuration for all sub-charts

### Default parameters (applied to all demo components)

| Property                               | Description                                                                               | Default                                              |
|----------------------------------------|-------------------------------------------------------------------------------------------|------------------------------------------------------|
| `default.env`                          | Environment variables added to all components                                             | Array of several OpenTelemetry environment variables |
| `default.envOverrides`                 | Used to override individual environment variables without re-specifying the entire array. | `[]`                                                 |
| `default.image.repository`             | Demo components image name                                                                | `otel/demo`                                          |
| `default.image.tag`                    | Demo components image tag (leave blank to use app version)                                | `nil`                                                |
| `default.image.pullPolicy`             | Demo components image pull policy                                                         | `IfNotPresent`                                       |
| `default.image.pullSecrets`            | Demo components image pull secrets                                                        | `[]`                                                 |
| `default.replicas`                     | Number of replicas for each component                                                     | `1`                                                  |
| `default.schedulingRules.nodeSelector` | Node labels for pod assignment                                                            | `{}`                                                 |
| `default.schedulingRules.affinity`     | Man of node/pod affinities                                                                | `{}`                                                 |
| `default.schedulingRules.tolerations`  | Tolerations for pod assignment                                                            | `[]`                                                 |
| `default.securityContext`              | Demo components container security context                                                | `{}`                                                 |
| `serviceAccount.annotations`           | Annotations for the serviceAccount                                                        | `{}`                                                 |
| `serviceAccount.create`                | Whether to create a serviceAccount or use an existing one                                 | `true`                                               |
| `serviceAccount.name`                  | The name of the ServiceAccount to use for demo components                                 | `""`                                                 |

### Component parameters

The OpenTelemetry demo contains several components (microservices). Each
component is configured with a common set of parameters. All components will
be defined within `components.[NAME]` where `[NAME]` is the name of the demo
component.

> **Note**
> The following parameters require a `components.[NAME].` prefix where `[NAME]`
> is the name of the demo component


| Parameter                               | Description                                                                                                | Default                                                       |
|-----------------------------------------|------------------------------------------------------------------------------------------------------------|---------------------------------------------------------------|
| `enabled`                               | Is this component enabled                                                                                  | `true`                                                        |
| `useDefault.env`                        | Use the default environment variables in this component                                                    | `true`                                                        |
| `imageOverride.repository`              | Name of image for this component                                                                           | Defaults to the overall default image repository              |
| `imageOverride.tag`                     | Tag of the image for this component                                                                        | Defaults to the overall default image tag                     |
| `imageOverride.pullPolicy`              | Image pull policy for this component                                                                       | `IfNotPresent`                                                |
| `imageOverride.pullSecrets`             | Image pull secrets for this component                                                                      | `[]`                                                          |
| `service.type`                          | Service type used for this component                                                                       | `ClusterIP`                                                   |
| `service.port`                          | Service port used for this component                                                                       | `nil`                                                         |
| `service.nodePort`                      | Service node port used for this component                                                                  | `nil`                                                         |
| `service.annotations`                   | Annotations to add to the component's service                                                              | `{}`                                                          |
| `ports`                                 | Array of ports to open for deployment and service of this component                                        | `[]`                                                          |
| `env`                                   | Array of environment variables added to this component                                                     | Each component will have its own set of environment variables |
| `envOverrides`                          | Used to override individual environment variables without re-specifying the entire array                   | `[]`                                                          |
| `replicas`                              | Number of replicas for this component                                                                      | `1` for ffsPostgres, kafka, and redis ; `nil` otherwise       |
| `resources`                             | CPU/Memory resource requests/limits                                                                        | Each component will have a default memory limit set           |
| `schedulingRules.nodeSelector`          | Node labels for pod assignment                                                                             | `{}`                                                          |
| `schedulingRules.affinity`              | Man of node/pod affinities                                                                                 | `{}`                                                          |
| `schedulingRules.tolerations`           | Tolerations for pod assignment                                                                             | `[]`                                                          |
| `securityContext`                       | Container security context to define user ID (UID), group ID (GID) and other security policies             | `{}`                                                          |
| `podSecurityContext`                    | Pod security context to define user ID (UID), group ID (GID) and other security policies                   | `{}`                                                          |
| `podAnnotations`                        | Pod annotations for this component                                                                         | `{}`                                                          |
| `ingress.enabled`                       | Enable the creation of Ingress rules                                                                       | `false`                                                       |
| `ingress.annotations`                   | Annotations to add to the ingress rule                                                                     | `{}`                                                          |
| `ingress.ingressClassName`              | Ingress class to use. If not specified default Ingress class will be used.                                 | `nil`                                                         |
| `ingress.hosts`                         | Array of Hosts to use for the ingress rule.                                                                | `[]`                                                          |
| `ingress.hosts[].paths`                 | Array of paths / routes to use for the ingress rule host.                                                  | `[]`                                                          |
| `ingress.hosts[].paths[].path`          | Actual path route to use                                                                                   | `nil`                                                         |
| `ingress.hosts[].paths[].pathType`      | Path type to use for the given path. Typically this is `Prefix`.                                           | `nil`                                                         |
| `ingress.hosts[].paths[].port`          | Port to use for the given path                                                                             | `nil`                                                         |
| `ingress.additionalIngresses`           | Array of additional ingress rules to add. This is handy if you need to differently annotated ingress rules | `[]`                                                          |
| `ingress.additionalIngresses[].name`    | Each additional ingress rule needs to have a unique name                                                   | `nil`                                                         |
| `command`                               | Command & arguments to pass to the container being spun up for this service                                | `[]`                                                          |
| `mountedConfigMaps[].name`              | Name of the Volume that will be used for the ConfigMap mount                                               | `nil`                                                         |
| `mountedConfigMaps[].mountPath`         | Path where the ConfigMap data will be mounted                                                              | `nil`                                                         |
| `mountedConfigMaps[].subPath`           | SubPath within the mountPath. Used to mount a single file into the path.                                   | `nil`                                                         |
| `mountedConfigMaps[].existingConfigMap` | Name of the existing ConfigMap to mount                                                                    | `nil`                                                         |
| `mountedConfigMaps[].data`              | Contents of a ConfigMap. Keys should be the names of the files to be mounted.                              | `{}`                                                          |
| `initContainers`                        | Array of init containers to add to the pod                                                                 | `[]`                                                          |
| `initContainers[].name`                 | Name of the init container                                                                                 | `nil`                                                         |
| `initContainers[].image`                | Image to use for the init container                                                                        | `nil`                                                         |
| `initContainers[].command`              | Command to run for the init container                                                                      | `nil`                                                         |

### Sub-charts

The OpenTelemetry Demo Helm chart depends on 1 sub-charts which is not enabled by default:
* Grafana

Parameters for each sub-chart can be specified within that sub-chart's
respective top level. This chart will override some of the dependent sub-chart
parameters by default. The overriden parameters are specified below.


#### Grafana

> **Note**
> The following parameters have a `grafana.` prefix.

| Parameter             | Description                                        | Default                                                              |
|-----------------------|----------------------------------------------------|----------------------------------------------------------------------|
| `enabled`             | Install the Grafana sub-chart                      | `true`                                                               |
| `grafana.ini`         | Grafana's primary configuration                    | Enables anonymous login, and proxy through the frontendProxy service |
| `adminPassword`       | Password used by `admin` user                      | `admin`                                                              |
| `rbac.pspEnabled`     | Enable PodSecurityPolicy resources                 | `false`                                                              |
| `datasources`         | Configure grafana datasources (passed through tpl) | Prometheus and Jaeger data sources                                   |
| `dashboardProviders`  | Configure grafana dashboard providers              | Defines a `default` provider based on a file path                    |
| `dashboardConfigMaps` | ConfigMaps reference that contains dashboards      | Dashboard config map deployed with this Helm chart                   |
