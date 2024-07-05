---
weight: 100
---

# Orchestrators

This step has the objective of registering the resource to the ICOS Agents local orchestrator. 
The step does not involve the installation and/or usage of any ICOS-specific components, 
so it is possible to refer to the orchestrators instructions for a detailed guide on how to 
register the resources.

## OCM Orchestrator
To join a cluster to the OCM Controller it is necessary to install clusteradm client 
on that cluster being registered. Detailed instructions can be found at 
[Register cluster installation official documentation](https://open-cluster-management.io/getting-started/installation/register-a-cluster/).

### Requirements
As part of managed cluster aggregation process the following prerequisites must be satisfied:

* Ensure kubectl and kustomize are installed.
* The managed clusters should be v1.11+.


## Nuvla Orchestrator
The procedure to deploy a NuvlaEdge to a node (here a node is a hardware device a bare-metal server, Virtual Machine, 
Raspberry Pi, Nvidia Jetson etc) is given by :anchor:[Nuvlaedge official documentation](https://docs.nuvla.io/nuvlaedge/installation/).

### Requirements
In order to install the NuvlaEdge and ensure its smooth execution over time, your device should have at least:

* 512MB of RAM;
* 2GB of free disk space;
* Supported CPU architectures: AArch32, AArch64 and x86_64;
* Docker Engine (version 18 or higher);
* docker CLI with compose command. Typically, installed along with Docker Engine;
* Network port 443: 
  * Outbound: Default HTTPS port. Used by NuvlaEdge to communicate with Nuvla via :anchor:[HTTPS](https://nuvla.io ).
* Network port 1194: 
  * Outbound: UDP connection to vpn.nuvlaedge.com in case VPN is decided to be used for communication 
    with the edge device managed by NuvlaEdge.

The full list of requirements can be found to the :anchor:[Nuvlaedge requierements](https://docs.nuvla.io/nuvlaedge/installation/requirements/).
