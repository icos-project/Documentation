# Deploy a new ICOS Controller

The section provides all the necessary information to install and deploy the ICOS Controller in a **Kubernetes cluster**.

## Hardware and Software Prerequisites
Since the ICOS Controller Suite consists of several modules that constitute the 
most important part of the system, the *minimum hardware requirements* are requested for a compatible host:

| Software               | RAM (GB) | CPU (cores x machine) | Storage (GB) |
| ---------------------- | ---------| --------------------- | ------------ |
| Kubernetes installed   | 2        | 2                     | 40           |

The installation furthermore requires full IP network connectivity between all machines 
that are part of the cluster (public or private network). 
For details on how to reach the required state of an installed Kubernetes distribution, 
the user can refer to the 
[Kubernetes documentation](https://kubernetes.io/docs/setup/production-environment/tools/kubeadm/install-kubeadm/).

Despite many distributions of Kubernetes, e.g. k3s, are compatible with ICOS, the current recommendation is limited to plain Kubernetes to host the controller suite with the following specifications:

- [Kubernetes 1.19 or higher](https://kubernetes.io/releases/)
- [Kubectl](https://kubernetes.io/docs/tasks/tools/install-kubectl-linux/)
- [Kustomize](https://kustomize.io/)
- [Clusteradm](https://github.com/open-cluster-management-io/clusteradm)
- [CSR Authentication support](https://en.wikipedia.org/wiki/Certificate_signing_request)
- [Orchestrator Control Plane](../ICOS Agent/Orchestrator Agents/controlplane.md)
- [Helm 3.7 (or newer versions)](https://github.com/helm/helm/releases).

!!! Note
	The suite is provided in the form of a helm chart and requires 
	[Helm 3.7 (or newer versions)](https://github.com/helm/helm/releases) to deploy the components.

## IAM Service Configuration

!!! info
    At the moment, this is a manual operation, that will be automated by the on-boarding process in the future releases.

Before deploying a new ICOS Controller the ICOS IAM service must configured creating a set of credentials that will be used by the new ICOS Controller.

1. First, create the following list of new OpenID Client (refer to [this documentation](../ICOS%20Core/IAM.md#create-an-openid-connect-client)):

    | Name   (can be customized) | Capabilities                                                                                         |
    | -------------------------- | ---------------------------------------------------------------------------------------------------- |
    | dynamic-policy-manager     | client authentication                                                                                |
    | matchmaker                 | client authentication, authorization                                                                 |
    | security-coordinator       | client authentication                                                                                |
    | telemetry-grafana          | client authentication, [ad hoc configuration](telemetry-dashboards.md#iam-configuration-for-grafana) |
    | topology-exporter          | client authentication, authorization                                                                 |


2. Then create a new user for the Shell Backend (follow the instructions [here](../ICOS%20Core/IAM.md#create-a-user).

Take note of all the clients and user ids and passwords because they are needed in the next step.

## ICOS Controller Suite installation
The ICOS Controller Suite is distributed as an Helm chart. To install it, first create a yaml file that contains the configuration values. This configure the release and instantiates all the Kubernetes components associated with the ICOS Controller.

Create a `values.yaml` file with the following content:

```yaml linenums="1"

global:

  # Configuration values for the new ICOS Controller to be deployed
  # for the clientId and clientSecret values use the values obtained 
  # in the IAM Configuration step.
  controller:
    id: <unique-id-for-the-controller>

    # These two values will define how the services will be exposed
    # customize to your preferences/needs
    url: https://controller-a.my-continuum.org/
    routing: host

    dpm:
      iamClientId: dynamic-policy-manager
      iamClientSecret: ********
    matchmaker:
      iamClientId: matchmaker
      iamClientSecret: ********
    securityCoordination:
      iamClientId: security-coordinator
      iamClientSecret: ********
    shellBackend:
      # the shell backend clientId and clientSecret MUST be the same used in the lighthouse configuration done for the ICOS Core
      iamClientId: lighthouse
      iamClientSecret: ********
      # user created in IAM Configuration step
      iamPassword: ********
      iamUser: ********
    telemetry:
      iamGrafanaClientId: telemetry-grafana
      iamGrafanaClientSecret: ********

  # Configuration values to identify the ICOS Core instance to which this ICOS Controller will register
  # These values should be the same used in the Helm values of the ICOS Core
  core:
    routing: host
    url: <url for the ICOS Core Suite>
    iam:
      publicKey: <IAM public Key>
      realm: <realm-name>
    # only needed if the ICOS CA is enabled in the ICOS Core
    ca:
      bundle: <CA Root Certificate Bundle in Base64>

icos-ingress-controller:
  enabled: <true|false>

```
!!! warning "Exposure"
    The values for `url`, `routing` and `icos-ingress-controller` must be carefully chosen since they will determin how the services of the ICOS Controller will be exposed. Make sure to **read carefully and fully understand** the guide to these values in the [Developers Guide](../../Developer/Suites/Exposure.md#icos-ingress-controller).

    In the provided example, services will be exposed using a LoadBalancer service routing the different services using DNS names.

The `global.core` values should match the ones used in the Helm release of the ICOS Core to which this new ICOS Controller will register to. 

To obtain the `global.core.iam.publicKey` value, follow [these instructions](../ICOS%20Core/IAM.md#retrieve-the-iam-public-key).

After the `values.yaml` file has been created, deploy the Helm release with:

```bash
helm install --namespace icos-system --create-namespace contrl1 oci://harbor.res.eng.it/icos/helm/icos-controller --values values.yaml
```

??? example "Development version"
	Note: In order to install versions not yet released, the url of the chart needs to be changed to 
	
	```bash
	oci://harbor.res.eng.it/icos-private/helm/icos-controller --values x.x.x-main.xxx
	```
	
	In addition, since unreleased versions are private, you need to login to the repository before 
	launching the install command: 
	```
	helm registry login harbor.res.eng.it/icos-private/helm and provide your credentials.
	```

## Customize the release

### Enable/Disable single components

Single components that are part of the ICOS Controller can be enabled/disabled using the fowlloing values:

```yaml
dynamic-policy-manager:
  enabled: false

shell-backend:
  enabled: false

aggregator:
  enabled: false

job-manager:
  enabled: false

job-manager-backend:
  enabled: false

telemetruum-hub:
  enabled: false

telemetruum-gateway:
  enabled: false

telemetruum-leaf:
  enabled: false

security-coordination-module:
  enabled: false

security-lomos-api2:
  enabled: false

intelligence-coordination:
  enabled: false

icos-metrics-export:
  enabled: false

matchmaker:
  enabled: false

dataclay:
  enabled: false

```

#### Persitence

By default, persistence is enabled and the components that needs to persist data will create a `PersistentVolumeClaim`. To control how the persistence is configured or disable it, use the following values (in the example provided, persistence is disabled for all components):

```yaml
job-manager-backend:
  primary:
    persistence:
      enabled: false

dynamic-policy-manager:
  mongodb:
    persistence:
      enabled: false

telemetruum-hub:
  grafana:
    persistence:
      enabled: false
  opensearch:
    master:
      persistence:
        enabled: false
  thanos:
    compactor:
      persistence:
        enabled: false
    receive:
      persistence:
        enabled: false
```

## Verify

To verify that the ICOS Controller has correctly registered itself to the continuum, it is possible to see if it is registered at the Lighthouse: 

```bash
> curl <lighthouse-url>/api/v3/controller/

[{"name":"<controller-id>","address":"<controller-url>"}]
```

!!! tips
    If the command fails with a `TLS` error, it might be related to the fact that the ICOS CA Root certificate is not trusted. Follow the instructions in the [ICOS CA section](../ICOS%20Core/CA.md) to install the certificate, or (not recommended) use the `-k` parameter to skip the certificate verification.


## Uninstall Chart
This removes all the Kubernetes components associated with the ICOS Controller and deletes the release.

```console
helm show values icos-controller
```

You may similarly use the above configuration commands on each chart dependency to see its configurations.

```console
helm uninstall [RELEASE_NAME]
```

