# Application Descriptor

## Introduction 

ICOS is a technology agnostic Meta Operating System. The aim of this document is to define the minimalist syntax required to provide ICOS with an agnostic Application Descriptor manifest to run on the ICOS system. 

## Application model

An application is made of a set of components and the interaction between them. Different entities that do not make sense separately (for instance a Deployment and the Service exposing it) should be defined as part of the same component. Therefore, they will run in the same resources (which could be a list of nodes in the same cluster). The service exposes the deployed component to facilitate communication with other components of the application.

!!! Note 
	Having two manifests with requirements for the same component does not make sense, because in fact we created the component so that we can group together entities that do not need computational resources with entities that need them. (Aka. Docker compose does this grouping and calls it a service).

## Overall structure of the application descriptor file

It has two mandatory sections: Components and Manifests. 

```console
name: uc4-app-emds
description: "the description"
components:
  - name: house-component
    type: kubernetes
    manifests:
      - name: house-manifest
manifests:
 # List of manifests in the specified format  
```

### Components

Components are application objects that can be executed at different nodes of the continuum. For each component, we need to specify a name, a list of manifests belonging to this component and the type of the orchestration engine that the manifests are prepared for. There are different cases to define the application components, each with specific manifests describing how to deploy the component on different orchestration engines. Below are examples for different cases.

Example 1: Two components with kubernetes type (single manifest each).

```console
name: my application
description: "The description of application"
components:
  - name: component-1
    type: kubernetes
    manifests:
      - name: comp-1-manifest

  - name: component-2
    type: kubernetes
    manifests:
      - name: comp-2-manifest
```

Example 2: Two components with docker type (single manifest each)

```console
name: my application
description: "The description of application"
components:
  - name: component-1
    type: docker
    manifests:
      - name: comp-1-manifest

  - name: component-2
    type: docker
    manifests:
      - name: comp-2-manifest
```

Example 3: Two components with mixed types (one kubernetes, one multiple)

```console
name: my application
description: "The description of application"
components:
  - name: component-1
    type: kubernetes
    manifests:
      - name: comp-1-manifest

  - name: component-2
    type: multiple
    manifests:
      - name: comp-2-manifest-kubernetes
      - name: comp-2-manifest-docker
```
Note: "multiple" means that both a kubernetes and docker manifests are provided for that specific component. However, if one of the component type is kubernetes the rest of the component will be deployed on kubernetes cluster. it is not allowed to mix components types.

Example 4: Two components with multiple type (two manifest each)

```console
name: my application
description: "The description of application"
components:
  - name: component-1
    type: multiple
    manifests:
      - name: comp-1-manifest-kubernetes
      - name: comp-1-manifest-docker

  - name: component-2
    type: multiple
    manifests:
      - name: comp-2-manifest-kubernetes
      - name: comp-2-manifest-docker
```
Note: In this case component-1 and component-2 are both flexible and can be deployed either on kubernetes or docker. 


The component section includes the list of components and two optional fields: [Requirements](#Requirements), [Policies](#Policies) and [Communication](#Communication)

#### Requirements

Requirements section specifies the requested resources for the application. It can apply to the entire application, in which case the section will appear at the same level as the components, or to individual components, in which case it will be specified under the component it applies to. 

!!! Note 
	At the moment the former option has not been implemented. 

Here is an example of a later option, where the requirements are specified inside each component.

Example : 
```console
name: my_application
description: "my app description"
components:
  - name: component-1
    type: kubernetes
    manifests:
      - name: comp-1-manifest
    requirements:
      architecture: intel
      cpu: 0.5
      memory: 200Mi
      nodelabel: 
        key: node-label-key
        values:
          - node-label-robot1
          - node-label-robot2
      devices: squat.ai/video
```
Accepted requirements are:

- **architecture**: intel/x86_64 or arm64/aarch64 

- **cpu**: number of cpu cores requested.

- **memory**: Amount of memory requested, the unit can be specified (following K8s syntax).

- **devices**: devices that are necessary to be present for the component to run.

- **nodelabel**: often the component is intended to run on specific nodes of the ICOS system (e.g., specific robots, specific houses, specific cars…). In this case a (key,value) pair can be specified to identify the nodes. The value can be a list of nodes, which implies that replicas of the component will be deployed in all of the specified nodes.


!!! Note 
	A note on defining the requirements for the ICOS header:
	In kubernetes, the smallest deployable units of computing are the Pods,so resources are allocated to Pods, whereas in ICOS 		the resource allocation unit is the component.

Since each container has its own resource needs, the user (application developer) needs to abstract (add up) the needs of each container (which can be part of a workload management controller such as Deployment, ReplicaSets , Job in K8s…) that belong to the same component. Similarly, if a component contains several manifests, each with its own list of requirements, the requirements for each manifest should be merged in the header, as shown in the example 5 below:

Example 5: **kubernetes manifests:**

```console
manifests:
  - name: comp-1-manifest-1
    manifest:
      apiVersion: apps/v1
      kind: Deployment
      metadata:
        name: busybox-1
        namespace: demo
      labels:
        app: busybox-1
      spec:
        selector:
          matchLabels:
            app: busybox-1
        template:
          metadata:
            labels:
              app: busybox-1
          spec:
            containers:
              - name: busybox-1-container
                image: busybox-1
                command: ["sleep", "100"]
                resources:
                  requests:
                    cpu: "0.5"
                    memory: "2Mi"
```

```console        
manifests:
  - name: comp-1-manifest-2
    manifest:
      apiVersion: apps/v1
      kind: Deployment
      metadata:
        name: busybox-2
        namespace: demo
      labels:
        app: busybox-2
      spec:
        selector:
          matchLabels:
            app: busybox-2
        template:
          metadata:
            labels:
              app: busybox-2
          spec:
            containers:
              - name: busybox-2-container
                image: busybox-2
                command: ["sleep", "100"]
                resources:
                  requests:
                    cpu: "0.5"
                    memory: "3Mi"
```          
  
Resulting ICOS header:

```console
components:
  - name: component-1
    type: kubernetes
    manifests:
      - name: comp-1-manifest-1
      - name: comp-1-manifest-2
    requirements:
      cpu: 1
      memory: 5Mi

manifests:
  - name: comp-1-manifest-1
    manifest:
      # The user provided kubernetes manifest (In the above example 5 busybox-1)
  - name: comp-1-manifest-2
    manifest:
      # The user provided kubernetes manifest (In the above example 5 busybox-2)
```
The implications of placing two manifests in the same component is that their requirements will be merged and a suitable target will be searched that matches ALL the requirements from both manifests. Therefore, it is understood that if two manifests can run apart from each other, they should belong to different components so that the matchmaking process has more options.

**Enforcement of requirements (Scheduling or runtime):**

When are requirements taken into account? In the current version, they are considered only at scheduling time. However, some of the requirements could also be monitored and enforced at runtime. In any case, the remediation action to take would be defined by ICOS (i.e. migrate).

#### Policies

Policies are more dynamic than requirements. They are rules that influence application deployment and thier behavior. Unlike requirements, which are static, policies are more dynamic and flexible. They can be defined inside a component or globally at the application level.

Syntax:
```console
policies:
  - name: policy-name
    type: policy-type
```

The rest of the dictionary entries depending on the type of policy. Supported types are:

- **Scheduler**: Specifies scheduling strategy.

- **Custom**: User-defined policies at runtime.

**Scheduling policy Example:**
The scheduling policy is checked during scheduling. It defines the scheduling strategy to use.

```console
policies:
  - name: scheduling-policy
    type: scheduler
    performance: 0.5
    energy: 0.3
    security:0.2
```
The scheduling policy prioritizes:

- Performance (50%)
- Energy efficiency (30%)
- Security (20%)

**Custom policy Example:**
The metric and remediation action are provided by the user. This policy applies at runtime.

```console
policies:
  - name: my-policy
    type: custom
    fromTemplate: compss-under-allocation
    remediation: scale-up
    variables:
      thresholdTimeSeconds: 120
      compssTask: provesOtel.example_task
```

#### Communication

This section specifies relationships between components of the application, as shown in example 6. It outlines how the interaction between components need to be defined in terms of communication between them (which is necessary if the components run in different clusters, so that ClusterLink can be set up). Each component needs to define its communication needs within its definition. This information will also be used during scheduling to find a more accurate allocation.

Each component has:
- **incoming**: which specify the service the component provide to another component.
- **outgoing**: which specify the service that the component depends on from another component.

Example 6:
```console
name: clusterlink
description: "application with component dependecies"
components:
  - name: component-1
    type: kubernetes
    manifests:
      - name: component-1-deployment
      - name: component-1-svc
    communication:
      - outgoing: component-2-svc
      - outgoing: component-3-svc

  - name: component-2
    type: kubernetes
    manifests:
      - name: component-2-deployment
      - name: component-2-svc
    communication:
      - incoming: component-1-svc

  - name: component-3
    type: kubernetes
    manifests:
      - name: component-3-deployment
      - name: component-3-svc
    communication:
      - incoming: component-1-svc

```
### Manifests

This section specifies the types of manifest that are supported: 

- **docker**: A docker compose manifest.

- **kubernetes**: A kubernetes manifest.

- **multiple**: Means both a kubernetes and dcompose manifests are provided, this gives more flexibility to the icos system to schedule the components in any resource, regardless of the orchestration engine. Whereas if a component is specified type kubernetes, it can only be scheduled at kubernetes clusters.

Example 7: **From docker compose manifest to ICOS header:**

**Docker compose manifest:**

```console
manifests:
  - name: comp-1-manifest
    manifest:
      version: '3.8'
      services:
        frontend:
          image: example/webapp
          deploy:
            resources:
              reservations:
                cpus: '0.5'
                memory: 200M
```

Resulting ICOS header:

```console
components:
  - name: component-1
    type: docker
    manifests:
      - name: comp-1-manifest
    requirements:
      cpu: 0.5
      memory: 200M

manifests:
  - name: comp-1-manifest
    manifest:
      # The user provided docker compose manifest as shown in example 7
```

!!! Note
    In the current ICOS implementation, it is not allowed to mix component types. That is, if one of the components is of type kubernetes, the rest should be either kubernetes or multiple. In this case, the application will be scheduled on kubernetes clusters. 


	




