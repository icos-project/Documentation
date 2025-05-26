# Getting Started

ICOS in an highly distributed system composed by a multiple nodes  deployed across the Cloud-Edge-IoT Continuum.

There are four different type of nodes, each of them providing different functionalities:

1. **ICOS Core**: it provides the basic Service Discovery, Security and Authentication/Authorization functionalities to the entire continuum. Only one node of this type should exist for each continuum
2. **ICOS Controllers**: they offers the main management, run-time, security and intelligence services
3. **ICOS Agents**: they control a subset of ICOS Workers and are responsible for offloading user's workloads in the most appropriate workers
4. **ICOS Workers**: these are the nodes where user's workloads (applications) will be executed. They can be:
    - a device/VM running a **Docker Engine**
    - a **Kubernetes** (or compatible, e.g. K3s) cluster made of one or more physical devices or VMs

A more in-depth explaination of the ICOS nodes can be found in the [ICOS Architecture](../Concepts/Architecture.md) section.

<figure markdown="span">
![An ICOS Continuum](./images/icos-continuum.png#only-light){ align=center }
![An ICOS Continuum](./images/icos-continuum-dark.png#only-dark){ align=center }
</figure>

## An ICOS Continuum

An ICOS Continuum starts with the installation of one ICOS Core node. Then, at least one ICOS Controller, one ICOS Agent and one ICOS Worker nodes should be installed in order to have a functional environment.

!!! tip "Nodes"

    The term **node** here, is used to identify the role that a particular infrastructure device (or VM) play in an ICOS Continuu and not to identify the infrastructural device itself. This means that it is possible to have multiple "nodes" in the same physical device. For instance, it is possible to deploy in the same Kubernetes cluster both the ICOS Core and an ICOS Controller node.  
    
    Theoretically, it is possible to install an entire ICOS Continuum in a single physical device (if capable enough)! :smile: However, in real-world deployments this make no sense, becuse the purpose of ICOS is to support the orchestration of workloads in a distributed set of infrastructures.

Each node can be installed and configured using a set of ICOS software distributions called **ICOS Suites** plus (in some cases) additional ICOS or third-party software needed to complete the configuration of the node.


## Create a minimal ICOS Continuum

!!! question "Navigation"
    This tutorial is **not a step-by-step guide**, but just provides an overivew of the full ICOS Continuum set-up with relevant links to follow to get each step completed. It should be used as **an outline to navigate and get oriented in the ICOS Administration Guide** to perform each needed for the administration of an ICOS Continuum.


In this section we will present an overview of all the steps necessary to deploy a minimal ICOS Continuum with only one worker.

### 1. Prepare the infrastructure

!!! note
    We present here just on possible deployment schema of an ICOS Continuum. The flexibility of the ICOS system allows for many more ways of deploying an ICOS Continuum.

We will imagine an organization that provides a customized device to their clients to be installed at their homes. The organization needs to control the devices at the client's homes to run specific workloads and upgrade the software.

The organization wants to establish an ICOS Continuum using:

- two Kubernetes clusters in a public cloud to install the ICOS Core and the ICOS Controller
- one Kubernetes cluster in a datacenter located in their headquarter office to install the ICOS Agent that will control all the devices in the same city
- one K3s cluster installed in each home device to install the ICOS Workers.

<figure markdown="span">
![Example ICOS Continuum infrastructure](./images/icos-continuum-2.png#only-light){ align=center }
![Example ICOS Continuum infrastructure](./images/icos-continuum-2-dark.png#only-dark){ align=center }
  <figcaption>Example ICOS Continuum infrastructure</figcaption>
</figure>

### 1. Install the ICOS Core

The ICOS Core is the first component to be installed because provides services used by all the other node types. It must be installed in a Kuberentes cluster reachable by, at least, all the ICOS Controllers.

The Kuberentes cluster uses the Cloud LoadBalancer of the provider to expose the services, so we can use the default values for the `icos-ingress-controller` component.

Following the instructions in the [Deploy the ICOS Core](./ICOS%20Core/icoscore.md) section, we prepare the following initial Helm values file:

```yaml
global:
  core:
    url: https://core.icos.example.org/
    routing: host

icos-ingress-controller:
  enabled: true
```

and run the `helm install` command (Step 1). We complete the configuration of the IAM Service (Step 2) and the upgrade of the ICOS Core (Steps 3 and 4).

At the end of all the installation steps we will have:

- the **IAM Service** running at `https://iam.core.example.org/`
- the **Lighthouse** running at `https://lighthouse.core.example.org/`


### 2. Install the ICOS Controller

After the ICOS Core is set-up, we can proceed with the installation of the ICOS Controller. We will install it in the same Cloud Provider, but in a different Kubernetes cluster.

Following the instructions in the [Deploy a new ICOS Controller](./ICOS%20Controller/installation.md) section, we first create the necessary configuration (clients and users) in the IAM service connecting to it at `https://iam.core.example.org/`. Then, we prepare the Helm values file as follow:

```yaml
global:

  controller:
    id: controller-a
    url: https://controller-a.icos.example.org/
    routing: host

    # --- security values taken from the IAM --- 
    dpm:
      iamClientId: ********
      iamClientSecret: ********
    matchmaker:
      iamClientId: ********
      iamClientSecret: ********
    securityCoordination:
      iamClientId: ********
      iamClientSecret: ********
    shellBackend:
      iamClientId: lighthouse
      iamClientSecret: ********
      # user created in IAM Configuration step
      iamPassword: ********
      iamUser: ********
    telemetry:
      iamGrafanaClientId: ********
      iamGrafanaClientSecret: ********
    # ----- 
    
  # --- configuration of the ICOS Core --- 
  core:
    url: https://core.icos.example.org/
    routing: host
    ca:
      bundle: LS0tLS1CRUdJTiBDRVJUSUZJQ0FURS0tLS0tCk1JSUJ3ekNDQVdtZ0F3SUJBZ0lRZmpnT3ozZDNDUDJtc1hLSEx1czJ3akFLQmdncWhrak9QUVFEQWpCQU1Sb3cKR0FZRFZRUUtFeEZUZEdWd0lFTmxjblJwWm1sallYUmxjekVpTUNBR0ExVUVBeE1aVTNSbGNDQkRaWEowYVdacApZMkYwWlhNZ1VtOXZkQ0JEUVRBZUZ3MHlOVEF4TVRrd09UUXpNekphRncwek5UQXhNVGN3T1RRek16SmFNRUF4CkdqQVlCZ05WQkFvVEVWTjBaWEFnUTJWeWRHbG1hV05oZEdWek1TSXdJQVlEVlFRREV4bFRkR1Z3SUVObGNuUnAKWm1sallYUmxjeUJTYjI5MElFTkJNRmt3RXdZSEtvWkl6ajBDQVFZSUtvWkl6ajBEQVFjRFFnQUUycWdZdFkSbIzPkPwMIclJFmNz6DqCEQXIvTjtQXJ6vPbeasdij79lrrM0JrU1FsTVRINXlhdkR1Qm1kYjNSYUNvOS9ZeWFZY3FYCk1nMUY0MHVyeVFSaHpxTkZNRU13RGdZRFZSMFBkSbIzPkPwMIclJFmNz6DqCEQXIvTjtQXJ6vPbeasdij79lrrkFmOEMKQVFFd0hRWURWUjBPQkJZRUZEaWFMdFZqOU00ZWFsUFBkVTVNRlNCQXBia3hNQW9HQ0NxR1NNNDlCQU1DQTBnQQpNRVVDSVFEWWVHb1NrWDdKSU1xRnlUMFJIcFlMUWxRNXFRRVBjOUVUNGhnRkt5S3drd0lnR3hLK3RVRTdKYi9TClNzQS9jNG9rWnBtU2RXbG5WTmVUQXRReGxkRjlYdk09Ci0tLS0tRU5EIENFUlRJRklDQVRFLS0tLS0K
      issuerKid: kSbIzPkPwMIclJFmNz6DqCEQXIvTjtQXJ6vPbeasdij
      issuerPassword: 1LD0faPQ1Vg2a6qSlfa1AQZktoSFPua0
    iam:
      # IAM public key (see IAM documentation to see how to retrieve it)
      publicKey: MIIBIjANBgkqhkiG9w0BAQEFAAOCAQ8AMIIBCgKCAQEAt04EcCgb1fk2Cre3dAcNhEeVmkrGeQvTdHCYjL263nyK3eCv8eeHPRLxJkPOIoE8/tSSEewOzWuSCMHceNX/Kox9D99LLde0mncpcIT54brLLCKHEa1fEJwpI9qO4LK4MbSyc5PMg+s4ekbF4ujma9jrNDDIME1n3fa4wYuNTiS6bZ0/paR7vJD2t2jeNH9lixVZQvF9kETkt2VYjEL5yX7NDn61n6oVYGNBc6tEOAp9Y3W6X6Ht2gMINm5sWTXr/yPcNshJms3PGUelZAQsGI8kL58dvnekZq29618OZGO8ya6xdcmuTqxe5srlQDA0OsEwGSvq/eH9NRR4ExMwIDAQAB
      realm: icos-continuum
    lighthouse:
      iamClientSecret: KwuRE60Gd0jHmHDpxK9wZ0Dq13B7gf2W
  # ----- 

icos-ingress-controller:
  enabled: true
```

In this Helm values file:

- all `global.core` values MUST match the ones created in the previous step to deploy the ICOS Core. This is necessary for this ICOS Controller to discover and register the correct ICOS Core instance 
- the `global.core.iam.publicKey` value MUST be retrieved from the IAM Service following [these instructions](./ICOS%20Core/IAM.md#retrieve-the-iam-public-key)
- all the security values under `global.controller` MUST match the ones from the IAM Service configuration

Finally we can run the `helm install` command and the ICOS Controller will be deployed.

```bash
helm install --namespace icos-system --create-namespace contrl1 oci://harbor.res.eng.it/icos/helm/icos-controller --values values.yaml
```

After the installation:

- the **Shell Backend** running at `https://shell.controller-a.icos.example.com/`

To verify that the ICOS Controller has correctly registered itself to the continuum we can run: 

```bash
> curl https://lighthouse.core.example.org/api/v3/controller/

[{"name":"controller-a","address":"shell.controller-a.icos.example.org"}]
```


### 3. Install the ICOS Agent

After the installation of the ICOS Controller we can proceed with the installation of the ICOS Agent that will register to that controller. The installation will be done in a Kubernetes cluster running in the our organization office. The services exposed by the Agent will need to be reachable by all the ICOS Workers that should be controlled by this agent.

#### Orchestrator installation
The first choice is to decide the orchestrator to use. At the moment OCM or Nuvla are supported. To more information on how to make this choice, consult the [Orchestrators section](./ICOS%20Agent/Orchestrators/index.md). 

Imagining that we selected OCM as orchestrator, we proceeed with the installation of the [OCM Control Plane](./ICOS%20Agent/Orchestrators/controlplane.md) in the cluster.

#### ICOS Agent Suite installation

In the same cluster where the orchestrator has been installed, we are now going to install the ICOS Agent software. Unlike for the previous nodes, here we don't have a Cloud LoadBalancer but we just have a single DNS name (`icos-agent.acme.com`) already configured to direct the traffic to our traffic. In addition, we want to deploy the services in a not standard port (`31500`).

Following the instructions in the **[Deploy a new ICOS Agent](./ICOS%20Agent/installation.md)** section, we first create the necessary configuration (clients and users) in the IAM service connecting to it at `https://iam.core.example.org/`. Then, we prepare the Helm values file as follow:

```yaml
global:
  agent:
    id: agent-1
    url: https://icos-agent.acme.com:31500/
    routing: path

    # --- security values taken from the IAM ---  
    ocmDM:
      iamClientId: ****
      iamClientSecret: ****
    # ----- 

  # --- configuration of the ICOS Controller --- 
  controller:
    url: https://controller-a.icos.example.org/
    routing: host
  # ----- 

  # --- configuration of the ICOS Core --- 
  core:
    url: https://core.icos.example.org/
    routing: host
    ca:
      bundle: LS0tLS1CRUdJTiBDRVJUSUZJQ0FURS0tLS0tCk1JSUJ3ekNDQVdtZ0F3SUJBZ0lRZmpnT3ozZDNDUDJtc1hLSEx1czJ3akFLQmdncWhrak9QUVFEQWpCQU1Sb3cKR0FZRFZRUUtFeEZUZEdWd0lFTmxjblJwWm1sallYUmxjekVpTUNBR0ExVUVBeE1aVTNSbGNDQkRaWEowYVdacApZMkYwWlhNZ1VtOXZkQ0JEUVRBZUZ3MHlOVEF4TVRrd09UUXpNekphRncwek5UQXhNVGN3T1RRek16SmFNRUF4CkdqQVlCZ05WQkFvVEVWTjBaWEFnUTJWeWRHbG1hV05oZEdWek1TSXdJQVlEVlFRREV4bFRkR1Z3SUVObGNuUnAKWm1sallYUmxjeUJTYjI5MElFTkJNRmt3RXdZSEtvWkl6ajBDQVFZSUtvWkl6ajBEQVFjRFFnQUUycWdZdFkSbIzPkPwMIclJFmNz6DqCEQXIvTjtQXJ6vPbeasdij79lrrM0JrU1FsTVRINXlhdkR1Qm1kYjNSYUNvOS9ZeWFZY3FYCk1nMUY0MHVyeVFSaHpxTkZNRU13RGdZRFZSMFBkSbIzPkPwMIclJFmNz6DqCEQXIvTjtQXJ6vPbeasdij79lrrkFmOEMKQVFFd0hRWURWUjBPQkJZRUZEaWFMdFZqOU00ZWFsUFBkVTVNRlNCQXBia3hNQW9HQ0NxR1NNNDlCQU1DQTBnQQpNRVVDSVFEWWVHb1NrWDdKSU1xRnlUMFJIcFlMUWxRNXFRRVBjOUVUNGhnRkt5S3drd0lnR3hLK3RVRTdKYi9TClNzQS9jNG9rWnBtU2RXbG5WTmVUQXRReGxkRjlYdk09Ci0tLS0tRU5EIENFUlRJRklDQVRFLS0tLS0K
      issuerKid: kSbIzPkPwMIclJFmNz6DqCEQXIvTjtQXJ6vPbeasdij
      issuerPassword: 1LD0faPQ1Vg2a6qSlfa1AQZktoSFPua0
    iam:
      # IAM public key (see IAM documentation to see how to retrieve it)
      publicKey: MIIBIjANBgkqhkiG9w0BAQEFAAOCAQ8AMIIBCgKCAQEAt04EcCgb1fk2Cre3dAcNhEeVmkrGeQvTdHCYjL263nyK3eCv8eeHPRLxJkPOIoE8/tSSEewOzWuSCMHceNX/Kox9D99LLde0mncpcIT54brLLCKHEa1fEJwpI9qO4LK4MbSyc5PMg+s4ekbF4ujma9jrNDDIME1n3fa4wYuNTiS6bZ0/paR7vJD2t2jeNH9lixVZQvF9kETkt2VYjEL5yX7NDn61n6oVYGNBc6tEOAp9Y3W6X6Ht2gMINm5sWTXr/yPcNshJms3PGUelZAQsGI8kL58dvnekZq29618OZGO8ya6xdcmuTqxe5srlQDA0OsEwGSvq/eH9NRR4ExMwIDAQAB
      realm: icos-continuum
    lighthouse:
      iamClientSecret: KwuRE60Gd0jHmHDpxK9wZ0Dq13B7gf2W
  # ----- 

ocm-descriptor:
  enabled: true

# use NodePort
icos-ingress-controller:
  enabled: true
  nginx-ingress-controller:
    service:
      type: NodePort
      nodePorts:
        https: 31500
```

In this Helm values file:

- all `global.core` values MUST match the ones created in the previous step to deploy the ICOS Core. This is necessary for this ICOS Agent to set-up the HTTPS certificate validation and issuing
- the `global.core.iam.publicKey` value MUST be retrieved from the IAM Service following [these instructions](./ICOS%20Core/IAM.md#retrieve-the-iam-public-key)
- the `global.controller.url` and the `global.controller.routing` values MUST match the ones used in the previous step for the ICOS Controller installation to let this ICOS Agent discover and connect to the correct ICOS Controller
- the values under `global.agent.ocmDM` MUST match the ones from the IAM Service configuration
- the values under `icos-ingress-controller` configure the exposure of the services on the port `31500`. The port number MUST match the one in the `global.agent.url` value!

Finally we can run the `helm install` command and the ICOS Agent will be deployed:

```bash
helm install --namespace icos-system --create-namespace agent1 oci://harbor.res.eng.it/icos/helm/icos-agent --values values.yaml
```

The ICOS Agent exposes the [Telemetry Gateway](../Concepts/Functionalities/observability.md) service used by the ICOS Workers to register to this ICOS Agent:

- the **Telemetry Gateway** will be running at `https://icos-agent.acme.com:31500/telemetry`


### 4. Install the ICOS Worker

The last component to be installed is the ICOS Worker. It will connect to the ICOS Agent and will a) send its telemetry data to the Agent and b) receive and execute the user's workloads from the Agent. The device where the worker will be install run a K3s installation and can reach the ICOS Agent (although the device itself is not exposed to the Internet).

The installation can be **broken in two parts**: the installation of the orchestrator agent and the installation of the ICOS software

#### Installation of the orchestrator agent

Since our ICOS Agent is using the OCM orchestrator we will have to install the OCM Agent and connect it to the orchestrator in the agent.

Following the [instructions in the OCM section](./ICOS%20Worker/Orchestrator%20Agents/openclustermanagementdeployment.md), we:

1. connect to the ICOS Agent cluster and generate a join token
2. use the join token in the Worker cluster to install the OCM Agent and send a joining request
3. connect again to the ICOS Agent cluster and accpet the joining request

Similar instructions would have been followed in case we were using [Nuvla](./ICOS%20Worker/Orchestrator Agents/Nuvlaedgedeployment.md).

#### ICOS software installation

Now we can deploy the ICOS Worker Suite that will be responsible for installing the required ICOS software (e.g. the Telemetry module for sending telemetry data to the ICOS Continuum). 

Folloing the guide in the [ICOS Worker Suite section](./ICOS%20Worker/workersuite.md), we prepare an Helm values file like this:

```yaml
global:
  agent:
    url: https://icos-agent.acme.com:31500/
    routing: path
  core:
    ca:
      bundle: LS0tLS1CRUdJTiBDRVJUSUZJQ0FURS0tLS0tCk1JSUJ3ekNDQVdtZ0F3SUJBZ0lRZmpnT3ozZDNDUDJtc1hLSEx1czJ3akFLQmdncWhrak9QUVFEQWpCQU1Sb3cKR0FZRFZRUUtFeEZUZEdWd0lFTmxjblJwWm1sallYUmxjekVpTUNBR0ExVUVBeE1aVTNSbGNDQkRaWEowYVdacApZMkYwWlhNZ1VtOXZkQ0JEUVRBZUZ3MHlOVEF4TVRrd09UUXpNekphRncwek5UQXhNVGN3T1RRek16SmFNRUF4CkdqQVlCZ05WQkFvVEVWTjBaWEFnUTJWeWRHbG1hV05oZEdWek1TSXdJQVlEVlFRREV4bFRkR1Z3SUVObGNuUnAKWm1sallYUmxjeUJTYjI5MElFTkJNRmt3RXdZSEtvWkl6ajBDQVFZSUtvWkl6ajBEQVFjRFFnQUUycWdZdFkSbIzPkPwMIclJFmNz6DqCEQXIvTjtQXJ6vPbeasdij79lrrM0JrU1FsTVRINXlhdkR1Qm1kYjNSYUNvOS9ZeWFZY3FYCk1nMUY0MHVyeVFSaHpxTkZNRU13RGdZRFZSMFBkSbIzPkPwMIclJFmNz6DqCEQXIvTjtQXJ6vPbeasdij79lrrkFmOEMKQVFFd0hRWURWUjBPQkJZRUZEaWFMdFZqOU00ZWFsUFBkVTVNRlNCQXBia3hNQW9HQ0NxR1NNNDlCQU1DQTBnQQpNRVVDSVFEWWVHb1NrWDdKSU1xRnlUMFJIcFlMUWxRNXFRRVBjOUVUNGhnRkt5S3drd0lnR3hLK3RVRTdKYi9TClNzQS9jNG9rWnBtU2RXbG5WTmVUQXRReGxkRjlYdk09Ci0tLS0tRU5EIENFUlRJRklDQVRFLS0tLS0K
```
Then we run the `helm install` command:

```bash
helm install --namespace icos-system --create-namespace wrk1 oci://harbor.res.eng.it/icos/helm/icos-worker --values values.yaml
```

We can add some **host labels** to the node following the instructions in the [Host Configuration section](../Developer/Components/telemetruum/Telemetruum Leaf/host-configuration.md).

To verify that the telemetry from the ICOS Worker is correctly "flowing", it is possible to connect to the [Telemetry Dashboards](./ICOS%20Controller/telemetry-dashboards.md#icos-agent-view-dashboard) to verify that the worker appears correctly. 

Other **optional components** not included in the ICOS Worker Suite (e.g. Wazuh agent for seurity assessment) can be deployed following the corresponding guides in the [ICOS Worker section](./ICOS%20Worker/index.md).

### 5. Install the ICOS Shell

Last step before being able to use the ICOS Continuum is to install the ICOS Shell Client in order to conenct to the continuum and perform operations (e.g. deploy an application).

1. We download the latest version of the ICOS Shell Client from [https://github.com/icos-project/Shell/releases/](https://github.com/icos-project/Shell/releases/)
2. We [create a user in the IAM service](./ICOS%20Core/IAM.md#create-a-user)
3. We prepare the shell configuration file `config.yaml`:
   ```yaml
    lighthouse: lighthouse.core.icos.example.org
    keycloak:
        pass: <iam-password>
        user: <iam-username>
   ```
4. We [install the ICOS CA Root Certificate](./ICOS%20Core/CA.md#trust-the-icos-ca-root-certificate) in our system
5. We login to the ICOS Continuum
   ```bash
   ./icos-shell --config=config.yaml auth login
   ```
6. We verify that everything worked listing the resources available in our ICOS Continuum
   ```bash
   ./icos-shell --config=config.yaml get resource
   ```

!!! warning
    A known issue with the handling of HTTPS URLs in the ICOS Shell requires to have a different configuration in case you exposed the Controller using HTTPS. Please refer to [Troubleshooting section](./troubleshooting.md#connection-refused-exception-while-using-any-command) for more details.

Congratulations we finished the set-up of our minimal ICOS Continuum! :tada: :tada: We are now ready to use our ICOS Continuum following the tutorials in the [User Guide](../User/index.md).

### 6. If something goes wrong...

In case of errors, it is possible to refer to the [Troubleshooting section](./troubleshooting.md) to solve the most common issues that can happen during the creation and the management of an ICOS Continuum.