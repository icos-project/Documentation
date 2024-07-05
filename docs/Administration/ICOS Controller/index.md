---
weight: 2
---

# ICOS Controller
The ICOS Controller is responsible for managing the [ICOS continuum (Cloud-Edge-IoT)](https://www.icos-project.eu/). 
(keeping track of the topology and availability of the current system) 
and the run-time (launching and monitoring the execution of services on-demand)
 
Each ICOS Controller manages a set of Agents based on a proximity criteria; 
hence, ICOS Controllers are deployed on resource-rich computing facilities along 
the continuum to cover the whole geographical area.
The specific functionalities of the ICOS controller are further divided into two main components.

- **Continuun Manager**.
- **Runtime Manager**.


### Continuum Manager:
The Continuum Manager is responsible to:

- Keeping track of the current and future (forecast) state of the infrastructure and 
  their availability (Aggregator). 

- On-boarding new resources to the [continuum resource & cluster manager](../Computational Resources/index.md).
  
- Storing and analyzing the system collected [telemetry](../Computational Resources/telemetryagent.md).

### Run-time Manager:
The Run-time Manager is responsible to:

- Receiving the application execution request ([Shell Backend](../../Developer/Components/Shell/shell-backend))

- Managing the users application execution by finding the best infrastructure to execute ([Matchmaking](../../Developer/Components/match-making/README.md)) 
and deploying the application in the selected infrastructure ([JobManager](../../Developer/Components/job-manager/api)).

- Monitoring the execution of the application and, if there is any violation of policies, 
  then take remediation actions ([Telemetry Agent](../../Developer/Components/telemetry-agent/installation.md)).
