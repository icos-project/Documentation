<!---
status: draft
review:
  editor: BSC
  version: 0.2
  comment: reviewed the content. It can published and removed this able.
  history: ENG as reported in the issue try to add a zenoh example. ENG edits the 1st draft version as from the [ICOS home page](https://www.icos-project.eu/architecture)
--->

# Data Management

The Data Management Module is responsible for managing all data required in ICOS, as well as the efficient 
execution layer of data-based applications and services used in ICOS. Its main functionalities include:

- Data distribution across the continuum, taking advantage of the entire available infrastructure
- Smart data placement and dynamic adaptation to changes in the infrastructure during operation: devices joining or leaving, reorganizations, etc
- Seamless access to data in ICOS, regardless of the location (device or cloud) or nature of resource (in-motion or at-rest), by providing an integrated data platform spanning the whole continuum
- Minimization of data transfers to improve performance and trust, by exploiting near-data processing invarious types of devices.

An example is provided in the [Example Zenoh section](zenoh.md). 