# Deploy a new ICOS Agent

ICOS agents are distributed throughout the continuum. Each ICOS controller manages a set of Agents 
and keeps track of them within its scope. In the current release, an ICOS agent includes the deployment manager 
that deploys specifically for [OCM](https://ocm/) or [Nuvla](https://docs.nuvla.io/nuvla), and manages 
the deployment accordingly. 
The ICOS agent is responsible to:

- Executing and offloading users services.
- Managing runtime communication between other agents.
- Providing the infrastructure information during the onboarding process to the controller 
  (topology component).
- Evaluation of policies and remediation action (local DPM-IT2) .

# Deploy a new ICOS Agent

The ICOS Agent Suite is a package that contains multiple ICOS services that are needed to manage the agent. 
In this first release of ICOS, the only component contained in the ICOS Agent Suite is the Deployment Manager.

## Requirements
The ICOS Agent Suite can be deployed everywhere (also externally to the resources that constitute 
the ICOS Agent), even if it will usually be deployed *close to* or *inside* the resources 
that are controlled by the Agent for convenience.

In the case the ICOS Agent will use OCM as orchestrator, then the ICOS Agent Suite **must** be deployed in the same cluster where the OCM Control Plane is deployed.

The Agent's Deployment Manager will need a service account to authenticate itself at the Controller's Job Manager. This needs to be created manually before starting the installation process.

From a networking point of view, the **constraints** for the location of the ICOS Agent Suite 
is that from that location it is possible to connect to both the ICOS Controller and the local 
orchestrator, and that hosts of the Agent are able to connect to the ICOS Agent Suite location.

As a prerequisite, [Helm](Helm) must be installed on the machine where the installation is performed 
(follow the instructions in the [Helm's documentation](https://helm.sh/docs/) to get started).

## Installation
The ICOS Agent Suite is packaged as an Helm chart and can be deployed in any Kuberentes 
(or compatible) cluster. It includes support for both OCM and Nuvla orchestrators. 
The user needs to specify which one to use using the values ==**ocm-descriptor.enable**== and 
==**nuvla.enable**==.


### Step1 - Configure

Create a file named ***my-values.yaml*** with the following content (customize the values to match 
your deployment):

```yaml
global:
  icos:
    iam:
      publicKey: <IAM Public Key>
      realm: <IAM Realm>
      baseUrl: <IAM URL>
    jobManager:
      baseUrl: <Job Manager Endpoint>
    lighthouse:
      baseUrl: <Light House Endpoint>


# Configuration specific for OCM. Include only if the orchestrator is OCM
ocm-descriptor:
  enabled: <true|false>
  configMap:
    keycloakClientId: <DM Client Id>
    keycloakClientSecret: <DM Client Secret>


# Configuration specific for Nuvla. Include only if the orchestrator is Nuvla
nuvla:
  enabled: <true|false>
  apiKey: *****
  apiSecret: *****

```

### Step2 - Install

Run the Helm installation command:

```console
helm install --namespace icos-system icos-agent oci://harbor.res.eng.it/icos/helm/icos-agent --values my-values.yaml
```
