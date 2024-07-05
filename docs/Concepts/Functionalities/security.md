# Security Layer
The Security Layer is responsible for guaranteeing the security of ICOS users, resources, and
applications at all times. It includes modules for authentication and authorization operations in the
system, assess security of resources and applications and suggest mitigation actions, proactive discovery
of anomalous behaviours and verification of the compliance of resources and applications. Trust
(identity validation) and Privacy (anonymisation and encryption) are architecture-wide functionalities
and not specific modules. Specifically the module should be:


1. **Federated Identity Management**.
2. **Authentication, Authorization** and **Audit** capabilities. 
3. **Detection** of security issues and **mitigation** mechanisms (e.g. self-healing). 
4. **Support** for compliance frameworks.
5. **Trust** and **privacy**, as supported by several architectural components:
	- **Federated Identity Management** and **Access control** AuthT and AuthZ libraries will be made available for usage by third-parties.
	- **Audit module** will provide audit capabilities based on different standards, such as OpenID Connect, which will allow insight into data and compute flow throughout the continuum.
	- **Security Assessment** will be used to detect potential security issues and already recommend specific mitigation or recovery processes.
	- **Recovery module** will provide a catalogue of recovery processes that may be atomatically enforced.
	- **Anomaly Detection** mechanisms will be built on top of the Intelligence layer and will provide input to the Recovery and Security Assessment module.
	- **Privacy module** will provide a set of primitives that could be used for data transformation, such as anonymisation, and encryption.
	- **Trust module** will provide the framework for attesting that the environment has not been tampered.
	- **Compliance enforcement** module will enable detection of compliance problems and enforce infrastructural changes in order to enhance compliance based on the chosen target compliance framework.
 
