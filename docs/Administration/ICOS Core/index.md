---
weight: 1
---

# ICOS Core

The ICOS Core consist of on a set of logical components to better manage its complexity
and analyse its design [D2.2 - ICOS Design](https://www.icos-project.eu/files/deliverables/D2.2_ICOS_Design_v1.0.pdf). Two types of ICOS Nodes have been identified:

- the **[ICOS Controller](../ICOS Controller/index.md)** (responsible for managing the continuum and the run-time) and
- the **[ICOS Agent](../ICOS Agent/installation.md)** (responsible for executing offloaded users services).

The ICOS software (that will run on all the [ICOS Nodes](../../Concepts/glossary/) has been logically decomposed in multiple
layers ([concepts functionalities](../../Concepts/Functionalities/index.md)):

- The **Distributed Meta-Kernel Layer** responsible for the management of the Continuum and the
Runtime Manager which is responsible for the execution of user's applications.

- The **Security Layer** responsible for guaranteeing the security aspects in a proactive way for all
operations and at all levels: from devices and users authentication and authorisation to application
behaviour anomaly detection and compliance enforcement.

- The **Intelligence Layer** responsible to provide artificial intelligence capabilities to ICOS operations
and decisions like optimisation of application deployment and resources usage, suggestion of
recovery actions and predictive analysis of monitoring data.

- The **Data Management Layer**: a horizontal component that will be used by the other layers to store,
exchange, and process data within the system in a secure and distributed manner.

- The **ICOS Shell Layer**: responsible for exposing all the system functionalities to the users providing
graphical and command line interfaces, management, and DevOps tools. 

To explore ICOS Continuum read the following documentation:

<div class="grid cards" markdown>

-   :books:   __IAM Service__

    ---

    How to manage IAM service.

    [&rarr; IAM service management](IAM.md)


-   :material-clock-fast:  __New deploy ICOS system__

    ---

	How to deploy a new computational resources in the ICOS Continuum.
    
    [&rarr; ICOS Worker](../ICOS Worker/index.md)

</div>

