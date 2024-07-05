# Manage Users

## ICOS IAM Service
All operations in the ICOS System are expected to be authenticated and properly authorized before being
executed. The authentication and the authorization functionalities are
provided by the Identity and Access Management component. For this reason, this component is invoked
in almost all interactions between components and involved in the realization of all the functionalities.
Three cases are analysed to describe how:

1. Users' authentication and authorization are achieved.
2. Service-to-Service authentication and authorization is achieved.
3. Cross-Controller identities and authorization are achieved.

The users that will be managed by the IAM component are the Application Integrators that interact with ICOS for deploying and managing their
applications. This does not include neither the final users of applications nor the devices that are part of
the Cloud Continuum.
All these workflows rely on the [OAuth2.0 protocol](https://oauth.net/2/) for the communication between components. This
ensures the maximum security and trust levels at the state of the art and an easy implementation and
integration in existing or new components. 

