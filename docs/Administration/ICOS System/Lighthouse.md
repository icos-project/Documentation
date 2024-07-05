# Lighthouse
The Lighthouse [[1](https://www.icos-project.eu/files/deliverables/D2.2_ICOS_Design_v1.0.pdf)] 
is a new actor in the ICOS system whose main role is to be the known and fixed contact point, always available,
to be contacted during the node onboarding stage or in case of loss of connectivity with the system.
The main functionalities of the lighthouse are the following:

+ Keeps track of all active ICOS Controllers in the system.
+ When a Controller joins the system, it will inform the lighthouse.
+ When an Agent joins the system for the first time, and it does not know the address of any Controller,
  it will request a Controller to the lighthouse.
+ When an Agent re-joins the system, and it does not find the known Controller, it will request a
  Controller to the lighthouse.
+ When an ICOS Shell user opens a session in ICOS, he or she will request a Controller to the
 lighthouse.

## How to manage
To deploy a lighthouse instance, simply deploy the shell-backend container and 
mount a configuration file inside to `/app/config.yml`.
The config file has to define the following sections:

```yaml

lighthouse:
	controller_timeout: 60
keycloak:
  	server: KEYCLOAK_INSTANCE
  	client_id: KEYCLOAK_CLIENT_ID
  	client_secret: KEYCLOAK_CLIENT_SECRET
  	realm: KEYCLOAK_CLIENT_REALM
```

It can then be ran for example via a simple docker command:

```docker

docker run --name icos-lighthouse -p 8080:8080 -v ./config_stable.yml:/app/config.yml  -d REPOSITORY/icos-shell-backend:TAG

```

Given correct configuration, all controllers then add their addresses to the list in regular intervals.
There is only one lighthouse needed per ICOS installation, regardless of the number of controllers.

## Controller-Lighthouse registration service
This service runs as a sidecar to the [shell backend](../../../Developer/Components/shell-backend) and 
periodically refreshes the controller's address entry. It needs to be supplied with a set of environment 
variables to run:

```
CLIENTID=<id of the client configured in keycloak>
SECRET=<secret for client id>
REALM=<realm configured in keycloak>
PASSWORD=<password for the keycloak user>
USER=<keycloak user>
KEYCLOAK=https://<keycloak address.tld>
LIGHTHOUSE=http://<lighthouse address>/api/v3
ADV_ADDRESS=<controller address to advertise, including port>
ADV_NAME=<controller name to advertise>
```
