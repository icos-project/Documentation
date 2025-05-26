<!---
status: draft
review:
  editor: BSC
  version: 0.2
--->

# Data Management

The Data Management component is responsible for managing and enabling access to data across the different layers of ICOS. It optimizes performance by leveraging the architectural continuum characteristics.
As a transversal component within the ICOS architecture, it supports the diverse data needs of other layers. 
It ensures the transparency of the infrastructure's distribution and heterogeneity while providing 
efficient data access.

At a high level, the Data Management Layer addresses three key needs related to data:

- **Transport**: This includes data messaging, brokering, and managing data in motion.
- **Mutable Data Structures**: Handling data that requires concurrent access and modification.
- **Compute**: Supporting data-intensive computations that benefit from data locality.

Although these areas overlap to some extent, they effectively represent the core pillars the Data Management Layer provides to the ICOS project.


The Data Management Layer in ICOS will be fulfilled by two software blocks:

- [Zenoh]( https://zenoh.io/): A pub/sub/query protocol.An example is provided in the [Example Zenoh section](zenoh.md). 
- [dataClay](https://dataclay.bsc.es/): A distributed active object store.

More details about the techinical description are available at : [D4.2 Data management, Intelligence and Security Layers (IT-2)](https://www.icos-project.eu/deliverables/)



