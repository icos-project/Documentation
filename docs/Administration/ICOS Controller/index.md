---
weight: 2
---

# ICOS Controller
The ICOS Controller is responsible for managing the  [ICOS continuum (Cloud-Edge-IoT)](https://www.icos-project.eu/),
which includes tracking the topology and availability of the current system, as well as managing runtime operations, such as launching and monitoring the execution of services on demand.
Each ICOS Controller manages a set of Agents based on proximity criteria. 
As a result, ICOS Controllers are deployed on resource-rich computing facilities across the continuum to ensure comprehensive geographical coverage.
The specific functionalities of the ICOS Controller are divided into two main components:

- **[Continuum Manager](#continuum-manager)**.

- **[Runtime Manager](#run-time-manager)**.

### Continuum Manager

The Continuum Manager is responsible for:

- Tracking the current and forecasted state of the infrastructure and its availability (Aggregator). 

- Onboarding new resources to the [continuum resource & cluster manager](../ICOS Worker/index.md).
  
- Storing and analyzing the system collected [telemetry](../../Concepts/Functionalities/observability.md).

### Run-time Manager

The Run-time Manager is responsible for:

- Receiving application execution requests via the ([Shell Backend](../../Developer/Components/Shell/backend/)).

- Managing the users application execution by finding the best infrastructure to execute ([Matchmaking](https://github.com/icos-project/Match-Making/)) 
and deploying the application in the selected infrastructure ([JobManager](../../Developer/Components/Job%20Manager/jobmanager/)).

- Monitoring the execution of the application and, if there is any violation of policies, then take remediation actions 
([Telemetry Agent](https://github.com/icos-project/Telemetry-Agent)).
