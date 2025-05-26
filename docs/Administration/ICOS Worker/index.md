---
status: draft
review:
  editor: ENG, IBM
  version: 0.1

weight: 100
---

# ICOS Worker

This section explains how to set up and run the ICOS Worker within the ICOS Testbed. 

## How to run 

To onboard an edge device into the ICOS Continuum instance running in the ICOS Testbed, follow these steps:

1. [Prepare the edge infrastructure](preparededgeinfrastructures.md) by installing Kubernetes or Docker.

2. Deploy one of the supported [ICOS orchestrators agents](Orchestrator Agents/index.md) (OCM/Nuvla). Here it is possible to choose between two technologies

3. [Install ICOS Core Suites](workersuite.md).


!!! Note

    ICOS currently supports two different orchestrators:OCM or Nuvla.
    Both are available in the ICOS Testbed, and the choice of orchestrator depends on the specific requirements of each project.
    Please note that while both OCM and Nuvla are compatible with Kubernetes, only Nuvla supports Docker.

Additionally, other resources are available:

- Wazuh Agent  can be accessed here:[Wazuh Agent](wazuh_agent.md)


- ClusterLink can be accessed here:[Clusterlink](clusterlink.md)

