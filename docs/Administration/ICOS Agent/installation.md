# Install the ICOS Agent Suite

ICOS agents are distributed throughout the continuum. Each ICOS controller manages a set of Agents 
and keeps track of them within its scope. In the current release, an ICOS agent includes the deployment manager 
that deploys specifically for [OCM](../ICOS Worker/Orchestrator Agents/openclustermanagementdeployment.md) or [Nuvla](https://docs.nuvla.io/nuvla), and manages 
the deployment accordingly. 
The ICOS agent is responsible to:

- Executing and offloading users services.
- Managing runtime communication between other agents.
- Providing the infrastructure information during the onboarding process to the controller 
  (topology component).
- Evaluation of policies and remediation action (local DPM-IT2) .

# ICOS Agent Suite

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
The user needs to specify which one to use using the values `ocm-descriptor.enable` and 
`nuvla-dm.enable`.


### Step1 - Configure

Installing a new ICOS Agent requires several configuration values to be known before starting the installation. Most of those parameters are related to the ICOS Controller and Core configuration (endpoints, routing, secrets). This problem will be mitigated by a CLI, still under development, that will reduce the number of needed parameters.

Before starting the installation, create the following list of new OpenID Client in the continuum's IAM service (refer to [this documentation](../ICOS%20Core/IAM.md#create-an-openid-connect-client)):

| Name   (can be customized) | Capabilities                         |
| -------------------------- | ------------------------------------ |
| my-agent.dm                | client authentication, authorization |

Then, create a file named **values.yaml** with the following content (customize the values to match 
your deployment):

```yaml
global:

  # values to configure this Agent
  agent:
    id: <unique id for the agent>
    url: <base url or ip for exposing Agent services>
    routing: host|path|port

    # needed only if ocm-descriptor: true
    ocmDM:
      # client id and client secret from the OpenID Client created before
      iamClientId: ****
      iamClientSecret: ****

    # needed only if nuvla.enabled: true
    nuvlaDM:
      # client id and client secret from the OpenID Client created before
      iamClientId: ****
      iamClientSecret: ****
      # to be created in the Nuvla Portal
      nuvlaApiKey: ****
      nuvlaApiSecret: ****

  # values to inject the configuration of the Controller to which this Agent will register
  controller:
    url: <controller url>
    routing: host|path|port

  # values to inject the configuration of the ICOS Core of the Continuum to which this Agent belongs
  core:
    url: <core url>
    routing: host|path|port
    iam:
      publicKey: ****
      realm: ****
    ca:
      bundle: ****
      # needed only if the agent is using HTTPS to expose its services and TLS certificates needs to be created
      issuerKid: ****
      issuerPassword: ****
      
# Configuration specific for OCM. Include only if the orchestrator is OCM
ocm-descriptor:
  enabled: <true|false>


# Configuration specific for Nuvla. Include only if the orchestrator is Nuvla
nuvla:
  enabled: <true|false>

icos-ingress-controller:
  enabled: true
```

??? example "Example Nuvla Agent with HTTPS and host routing"
    A real example of a valid values to deploy an ICOS Agent that uses Nuvla as orchestrator is the following
    ```yaml
    global:
      agent:
        id: uc-agent-2
        nuvlaDM:
          iamClientId: uc-agent-2.dm
          iamClientSecret: ************
          nuvlaApiKey: ************
          nuvlaApiSecret: ************
        url: https://agent-2.icos-stable.10-160-3-240.sslip.io:31000/
        routing: host
      controller:
        url: https://controller-1.icos-stable.10-160-3-234.sslip.io:30000/
        routing: host
      core:
        routing: host
        url: https://core.icos-stable.10-160-3-234.sslip.io:30000/
        iam:
          publicKey: MIIBIjANBgkqhkiG9w0BAQEFAAOCAQ8AMIIBCgKCAQEA2lj6EIO30m2i2bVaVE9GUzotIBeUBlk5vUmQF1cFzR9Vk2jR8WOYOlq1RZtIIL/qXS8NG09rsosJ/HyvlRQXruIM25edGFPXEKxYIAFt5GsCjoxfKxNNJFwOkkrO9bS4dAkmCx5McxB1cr8T8/9GDtYiYkw9uuMCk8Kr7nPAjoB3PTgjWFDcORAozmCWjUYkyiMn3DDUkG2Er09N1QzjfrUgdPGMhC8aDEvnlOsMKOuywGyQ9YfPTcfR04jCe2GSlCTTPFLD8D6fDLl8AQlheFHKpNhCH0Nqo4+lNTzyZvERdvc8ac9yVUoyKJufBGndiLKAyNKQmQdK+XF3Id5FfQIDAQAB
          realm: icos-stable
        ca:
          bundle: LS0tLS1CRUdJTiBDRVJUSUZJQ0FURS0tLS0tCk1JSUJ3ekNDQVdtZ0F3SUJBZ0lRZmpnT3ozZDNDUDJtc1hLSEx1czJ3akFLQmdncWhrak9QUVFEQWpCQU1Sb3cKR0FZRFZRUUtFeEZUZEdWd0lFTmxjblJwWm1sallYUmxjekVpTUNBR0ExVUVBeE1aVTNSbGNDQkRaWEowYVdacApZMkYwWlhNZ1VtOXZkQ0JEUVRBZUZ3MHlOVEF4TVRrd09UUXpNekphRncwek5UQXhNVGN3T1RRek16SmFNRUF4CkdqQVlCZ05WQkFvVEVWTjBaWEFnUTJWeWRHbG1hV05oZEdWek1TSXdJQVlEVlFRREV4bFRkR1Z3SUVObGNuUnAKWm1sallYUmxjeUJTYjI5MElFTkJNRmt3RXdZSEtvWkl6ajBDQVFZSUtvWkl6ajBEQVFjRFFnQUUycWdZdFhsNwpQS1VUUHRjQ2w0MkltV0Q0RlEwclFHNXh1bVZkbkozM0JrU1FsTVRINXlhdkR1Qm1kYjNSYUNvOS9ZeWFZY3FYCk1nMUY0MHVyeVFSaHpxTkZNRU13RGdZRFZSMFBBUUgvQkFRREFnRUdNQklHQTFVZEV3RUIvd1FJTUFZQkFmOEMKQVFFd0hRWURWUjBPQkJZRUZEaWFMdFZqOU00ZWFsUFBkVTVNRlNCQXBia3hNQW9HQ0NxR1NNNDlCQU1DQTBnQQpNRVVDSVFEWWVHb1NrWDdKSU1xRnlUMFJIcFlMUWxRNXFRRVBjOUVUNGhnRkt5S3drd0lnR3hLK3RVRTdKYi9TClNzQS9jNG9rWnBtU2RXbG5WTmVUQXRReGxkRjlYdk09Ci0tLS0tRU5EIENFUlRJRklDQVRFLS0tLS0K

    nuvla-dm:
      enabled: true
    ```

??? example "Example OCM Agent with no HTTPS and port routing"
    ```yaml
    global:
      agent:
        id: uc-agent-3
        ocmDM:
          iamClientId: uc-agent-3.dm
          iamClientSecret: ************
        url: 10.160.3.300:32000
      controller:
        url: url: 10.160.3.100:32000
      core:
        url: 10.160.3.050:32000
        iam:
          publicKey: MIIBIjANBgkqhkiG9w0BAQEFAAOCAQ8AMIIBCgKCAQEA2lj6EIO30m2i2bVaVE9GUzotIBeUBlk5vUmQF1cFzR9Vk2jR8WOYOlq1RZtIIL/qXS8NG09rsosJ/HyvlRQXruIM25edGFPXEKxYIAFt5GsCjoxfKxNNJFwOkkrO9bS4dAkmCx5McxB1cr8T8/9GDtYiYkw9uuMCk8Kr7nPAjoB3PTgjWFDcORAozmCWjUYkyiMn3DDUkG2Er09N1QzjfrUgdPGMhC8aDEvnlOsMKOuywGyQ9YfPTcfR04jCe2GSlCTTPFLD8D6fDLl8AQlheFHKpNhCH0Nqo4+lNTzyZvERdvc8ac9yVUoyKJufBGndiLKAyNKQmQdK+XF3Id5FfQIDAQAB
          realm: icos-stable

    ocm-descriptor:
      enabled: true
    ```

The `url` and `routing` values recur multiple times in the values file and can be confusing. They are used to simplify the configuration and derive autoamtically the ICOS services endpoints. A more detailed explaination can be found in the [Developers Guide](../../Developer/Suites/Exposure.md).

The `icos-ingress-controller` component should be enabled only if a) the agent is configured to use a `host` or `path` routing AND 2) if there are no other instances of the same component in the cluster. A more detailed exaplination can be found in the [Developers Guide](../../Developer/Suites/Exposure.md#icos-ingress-controller)

### Step2 - Install

Run the Helm installation command:

```console
helm install --namespace icos-system --create-namespace agent1 oci://harbor.res.eng.it/icos/helm/icos-agent --values values.yaml
```

??? example "Development version"
	Note: In order to install versions not yet released, the url of the chart needs to be changed to 
	
	```bash
	oci://harbor.res.eng.it/icos-private/helm/icos-agent --values x.x.x-main.xxx
	```
	
	In addition, since unreleased versions are private, you need to login to the repository before 
	launching the install command: 
	```
	helm registry login harbor.res.eng.it/icos-private/helm and provide your credentials.
	```
