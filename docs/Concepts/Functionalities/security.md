# Security Layer
The Security Layer is responsible for guaranteeing the security of ICOS users, resources, and
applications at all times. It includes modules for authentication and authorization operations in the
system, assessing the security of resources and applications and suggesting remediation or mitigation actions, proactive discovery of anomalous behaviours and security-sensitive events, and verification of the compliance of resources and applications. Trust (identity validation) and Privacy (anonymization and encryption) are included as architecture-wide functionalities and not specific modules. 

Main functionalities of the Security Layer are:

1. **Security Layer Coordination API**, a unified interface for interacting with modules in the Security Layer;

2. **Authentication** and **Authorization**, along with AuthT and AuthZ libraries for usage by third-parties; 

3. **Audit** (performing light-audit security checks);

4. **Detection** of security issues and **mitigation** mechanisms (recommendations of specific mitigation or remediation actions); 

5. **Anomaly Detection** in the system and application;  

6. **Compliance detection** and **enforcement** mechanisms (recommendations of infrastructural changes in order to enhance compliance) and; 

7. **Trust** through using secure and trusted communication protocols and **Privacy** through anonymisation.