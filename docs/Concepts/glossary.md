# GLOSSARY

<table>
<tr><td style="background-color:blue;color:white;"><strong>Architecture</strong></td></tr>
<tr><td>An Architecture is an abstraction of the system that identifies its essential aspects like which is its intended use, 
its main elements and properties, the relationships between elements and between the system and the environment.
</td></tr>
<tr><td style="background-color:blue;color:white;"><strong>Architecture Description(AD)</strong></td></tr>
<tr><td>An Architecture Description (AD) is an artefact that documents the architecture of a system.An AD can take the form of one or more documents, models, diagrams, or simulations. 
An essential aspect for producing an AD is to know to whom the AD is of interest. 
</td></tr>
<tr style="background-color:blue;color:white;font-weight:bold"><td><strong><strong>Architectural View</strong></td></tr>
<tr><td>An Architectural View is a part of an AD that expresses the Architecture of the System from the perspective of one 
or more Stakeholders to address their specific concerns. They are composed of one or more View Components 
(e.g., diagrams, tables, sections of texts, interviews). This document presents a functionality view, a logical view, 
an operational view, a physical view and an implementation view.</td></tr>
<tr style="background-color:blue;color:white;font-weight:bold"><td><strong><strong>Cloud Continuum (CC)</strong>
</td></tr>
<tr><td>A group of resources (CPU, memory, network, storage, intelligence) managed seamlessly end-to-end that may span across different administrative/technology domains in multi-operator and multi-tenant settings.
</td></tr>
<tr style="background-color:blue;color:white;font-weight:bold"><td><strong><strong>Domain</strong></td></tr>
<tr><td>An infrastructure managed and controlled under the provisions of a single administration entity. Maybe further detailed depending on the complexity of the infrastructure topology , the underlying technologies and the NBI provided for third party management/orchestration.
</td></tr>
<tr style="background-color:blue;color:white;font-weight:bold"><td><strong><strong>ICOS Agent
</strong></td></tr>
<tr><td>The basic ICOS software component that needs to be installed on each node (infrastructure node) capable of executing external software and/or providing data and metadata.
</td></tr>
<tr style="background-color:blue;color:white;font-weight:bold"><td><strong><strong>ICOS Application Lifecycle Controller
</strong></td></tr>
<tr><td>Application Lifecycle Controller that manages a pool of resources across CC and is distributed across the ICOS controllers employing E/W interfaces. The Application Lifecycle Controller has the authority to control the lifecycle of the deployed Application/Service over the associated resources (i.e. the CC).
</td></tr>
<tr style="background-color:blue;color:white;font-weight:bold"><td><strong><strong>ICOS Application Template 
</strong></td></tr>
<tr><td>It is provided by ICOS Shell. It is the ICOS Application descriptor containing metadata and allows the definition of application components and deployment within the Cloud Continuum 
</td></tr>
<tr style="background-color:blue;color:white;font-weight:bold"><td><strong><strong>ICOS Controller
</strong></td></tr>
<tr><td>The ICOS component/software that is responsible for peering with ICOS Agents for providing control and lifecycle operations.
</td></tr>
<tr style="background-color:blue;color:white;font-weight:bold"><td><strong><strong>ICOS Element
</strong></td></tr>
<tr><td>Is the unitary resource available on a node (e.g. Computing, Storage, Network, Intelligence etc.). The basic, unitary ICOS Element is defined as the minimum set of resources defined as CPU/RAM/Storage (dynamically allocated by the native infrastructure management administration) encapsulated within HW or SW entities such as (VM, container or HW node). 
</td></tr>
<tr style="background-color:blue;color:white;font-weight:bold"><td><strong><strong>ICOS Instance 
</strong></td></tr>
<tr><td>A subset of the CC participating in the execution of a multi-component Application of a certain topology, resource requirements and constraints."
</td></tr>
<tr style="background-color:blue;color:white;font-weight:bold"><td><strong><strong>ICOS Networking
</strong></td></tr>
<tr><td>The ICOS will exploit available APIs and build logical connections as required for the realization of ICOS node interconnection or ICOS Instance. 
</td></tr>
<tr style="background-color:blue;color:white;font-weight:bold"><td><strong><strong>ICOS Node 
</strong></td></tr>
<tr><td>Any resource physical or virtual that is running either an ICOS Controller or Agent. 
</td></tr>
<tr style="background-color:blue;color:white;font-weight:bold"><td><strong><strong>ICOS Shell
</strong></td></tr>
<tr><td>A client distribution that is used to access the ICOS. The ICOS Shell runs externally to the ICOS Instance.
</td></tr>
<tr style="background-color:blue;color:white;font-weight:bold"><td><strong><strong>Restricted Capability Devices 
</strong></td></tr>
<tr><td>IoT device or infrastructure element (that can be resource-restricted) controlled through API provided by ICOS Agent on ICOS Nodes. The IoT device or infrastructure element could also be directly accessed via its control interface (e.g., API) i.e., not necessarily through the ICOS Node.
</td></tr>
<tr style="background-color:blue;color:white;font-weight:bold"><td><strong>System Use Case (SUC)</strong></td></tr>
<tr><td>A System Use Case (SUC) represents a single functionality that the system can provide to one or more actors that 
results in an observable result that is of some value for those actors. The term “System Use Case” used in this context is 
not related to the project’s “Use Cases.” The former, defined here,represents basic pieces of functionalities that 
the system provides; the latter identify the four project’s use cases described proposal that will be used to validate 
the ICOS System. When references to SUCs are used in this document, they are expressed with italic and underlined text.
</td></tr>
<tr style="background-color:blue;color:white;font-weight:bold"><td><strong>Stakeholder</strong></td></tr>
<tr><td>Stakeholders are entities that have an interest in the System. They can be individuals, groups, or organizations. Stakeholders use the AD to understand and analyse the system and as guidance for their activities. 
The AD should address all the interests (concerns) of all the stakeholders. However, doing it with a single model or diagram 
for a complex system like ICOS is impossible. For this reason, the AD is organized in multiple Architectural Views.
</td></tr>
</table>
