# Deploy a new ICOS Controller
The section provides all the necessary information to install and deploy 
the ICOS Controller in a **Kubernetes cluster**.

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
- [Orchestrator Control Plane](../Computational Resources/Orchestrators/index.md)
- [Helm 3.7 (or newer versions)](https://github.com/helm/helm/releases).

???note
	The suite is provided in the form of a helm chart and requires 
	[Helm 3.7 (or newer versions)](https://github.com/helm/helm/releases) to deploy the components.

## Installation
The ICOS Controller Suite is distributed as an Helm chart. 
To install it, first create a yaml file that contains the configuration values.
This creates the release and instantiates all the Kubernetes components 
associated with the ICOS Controller.

```yaml
global:
  external:
    host: <controller public ip>
  icos:
    controllerId: <controller unique name>
    iam:
      baseUrl: https://iam-url/
      realm: icos-prod1
      # used by ICOS services to verify JWT tokens. Get it from IAM Portal: Realm Settings > Keys > RS256 > Public Key
      publicKey: <IAM public key>
    lighthouse:
      baseUrl: http://lighthouse-url/

```

```console

helm install --namespace icos-system icos-controller oci://harbor.res.eng.it/icos/helm/icos-controller --values my-values.yaml

```

The Controller installation comprises the installation of multiple components, 
each of them with additional configuration values to be specified in the values file. The full list of configuration values is out of the scope of this section, more information on the composition of the 
ICOS Controller Suite and its configuration are provided in the deliverable [D5.1](https://www.icos-project.eu/files/deliverables/D5.1_Alpha_release_v1.0.pdf).

_See [configuration](#configuration) below._

_See [helm install](https://helm.sh/docs/helm/helm_install/) for command documentation._

## Components
This chart installs several additional charts corresponding to the components of the ICOS Controller

- **[shell-backend](https://production.eng.it/gitlab/icos/suites/icos-controller/-/tree/main/charts/shell-backend)**
- **[job-manager](https://production.eng.it/gitlab/icos/suites/icos-controller/-/tree/main/charts/shell-backend)**


## Uninstall Chart
This removes all the Kubernetes components associated with the ICOS Controller and deletes the release.

```console
helm uninstall [RELEASE_NAME]
```
_See [helm uninstall](https://helm.sh/docs/helm/helm_uninstall/) for command documentation._

## Configuration
See [Customizing the Chart Before Installing](https://helm.sh/docs/intro/using_helm/#customizing-the-chart-before-installing). To see all configurable options with detailed comments, visit the chart's [values.yaml](./values.yaml), or run these configuration commands:

```console
helm show values icos-controller
```

You may similarly use the above configuration commands on each chart [dependency](#dependencies) to see its configurations.
