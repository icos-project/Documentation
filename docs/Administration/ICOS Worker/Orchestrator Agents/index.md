---
weight: 1
---

# Orchestrators

This step has the objective of registering the resource to the ICOS Agents local orchestrator. 
The step does not involve the installation and/or usage of any ICOS-specific components, 
so it is possible to refer to the orchestrators instructions for a detailed guide on how to 
register the resources.
ICOS currently supports two different orchestrators: 

- [OCM](openclustermanagementdeployment.md) or 

- [Nuvla](Nuvlaedgedeployment.md).
 
In the ICOS testbed we provide both of them. 
It is up to the single project to choose the best orchestrator for her use cases. 
Please note that while both OCM and Nuvla can work with Kubernetes, only Nuvla supports Docker. 
So if in step 2, Docker was deployed the only available choice is Nuvla.

## OCM Orchestrator
To join a cluster to the OCM Controller it is necessary to install clusteradm client 
on that cluster being registered. 

### Requirements
As part of managed cluster aggregation process the following prerequisites must be satisfied:

* Ensure kubectl and kustomize are installed.
* The managed clusters should be v1.11+.


## Nuvla Orchestrator
The procedure to deploy a NuvlaEdge to a node (here a node is a hardware device a bare-metal server, Virtual Machine, 
Raspberry Pi, Nvidia Jetson etc) is given by [Nuvlaedge official documentation](https://docs.nuvla.io/nuvlaedge/installation/).

### Requirements
In order to install the NuvlaEdge and ensure its smooth execution over time, your device should have at least:

* 512MB of RAM;
* 2GB of free disk space;
* Supported CPU architectures: AArch32, AArch64 and x86_64;
* Docker Engine (version 18 or higher);
* docker CLI with compose command. Typically, installed along with Docker Engine;
* Network port 443: 
  * Outbound: Default HTTPS port. Used by NuvlaEdge to communicate with Nuvla via [HTTPS](https://nuvla.io/ui).
* Network port 1194: 
  * Outbound: UDP connection to vpn.nuvlaedge.com in case VPN is decided to be used for communication 
    with the edge device managed by NuvlaEdge.

The full list of requirements can be found to the[Nuvlaedge requierements](https://docs.nuvla.io/nuvlaedge/installation/requirements/).

After the installation of the Orchestrator (OCM or Nuvla),you can deploy the telemetry component.
 
