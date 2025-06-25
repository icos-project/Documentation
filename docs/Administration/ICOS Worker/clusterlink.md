# ClusterLink Installation Guide

The information here is based on [ClusterLink documentation](https://clusterlink.net/docs/v0.4/getting-started/users/).

## What is ClusterLink?

> ClusterLink simplifies the connection between application services that are located in different domains, networks, and cloud infrastructures.


## Install ClusterLink CLI on a machine
For each cluster where ClusterLink will run, install `clusterlink` on the master machine.

Run the installation script:

```
curl -L https://github.com/clusterlink-net/clusterlink/releases/download/v0.4.0/clusterlink.sh | sh -
```

Check the installation by running the command:

```
clusterlink –-version
```

## Create a ClusterLink fabric
This is a one-time procedure to build a ClusterLink fabric to connect multiple clusters.

Log in to the master machine on your first cluster.

Create a directory named `ClusterLink`.

From inside the `ClusterLink` directory, create the fabric’s certificate authority (CA) certificate and private key

```
clusterlink create fabric
```

This will create a fabric with the default name: `defaut_fabric`.
This command will create the CA files `cert.pem` and `key.pem` in a directory named `defaut_fabric`.
These files will be needed to create ClusterLink peers on each cluster where ClusterLink is installed.

## Connect a cluster to a ClusterLink fabric

Perform the following procedure on the master node of each cluster added to the fabric (including the first cluster).

Ensure you have a `ClusterLink` directory with subdirectory `default_fabric` containing files `cert.pem` and `key.pem`.

If necessary, copy these files from the original ClusterLink fabric installation.
```
--/ClusterLink
--/ClusterLink/default_fabric
--/ClusterLink/default_fabric/cert.pem
--/ClusterLink/default_fabric/key.pem
```
### Create a peer certificate

Choose `<peer_name>` to be `<ICOS Agent ID>-<ICOS Cluster Name>`.

Run the following commands from the `ClusterLink` directory.


```
clusterlink create peer-cert --name <peer_name>
```

Verify that the directory `default_fabric/<peer_name>` was created with files `cert.pem` and `key.pem` in that directory.
```
--/ClusterLink
--/ClusterLink/default_fabric
--/ClusterLink/default_fabric/cert.pem
--/ClusterLink/default_fabric/key.pem
--/ClusterLink/default_fabric/<peer-name>/cert.pem
--/ClusterLink/default_fabric/<peer-name>/key.pem
```

### Install ClusterLink Deployment

```
clusterlink deploy peer --name <peer_name> --ingress=NodePort --ingress-port=30443
```

Verify that the ClusterLink control and data plane components are running.

It may take a few seconds for the deployments to be successfully created.

```
kubectl rollout status deployment cl-controlplane -n clusterlink-system
kubectl rollout status deployment cl-dataplane -n clusterlink-system
```

Apply an access policy to allow other ClusterLink peers to connect.

```
kubectl apply -f policy.yaml
```

For `policy.yaml`, use the following:

```
apiVersion: clusterlink.net/v1alpha1
kind: AccessPolicy
metadata:
  name: allow-policy
  namespace: default
spec:
  action: allow
  from:
    - workloadSelector: {}
  to:
    - workloadSelector: {}
```

### Set up peers between clusters

See example [here](https://clusterlink.net/docs/v0.4/tutorials/iperf/#set-up-peers).

For each peer to which you want to connect, prepare a `peer-<peer-name>.yaml` file.

It may be easiest to copy some existing `peer*` files (scp from existing installation, `~/ClusterLink` directory) to the `ClusterLink` directory.
Install existing peers on new cluster and install new cluster as a peer on existing clusters that may use resources on the new cluster.

```
kubectl apply -f peer-xxxx.yaml
```

The peer-xxxx.yaml file looks like this:
```
apiVersion: clusterlink.net/v1alpha1
kind: Peer
metadata:
  name: <peer-name>
  namespace: clusterlink-system
spec:
  gateways:
    - host: <peer-address>
      port: 30443
```
The fields that must be adjusted per instance are the `name` and `host` fields.

### Export / Import

When running an application, define Export and Import of services between peers / clusters.
(This should be done by the Deployment Manager.)
See example [here](https://clusterlink.net/docs/v0.4/tutorials/iperf/#export-the-iperf-server-endpoint).

To export a service on a cluster, create a `service-export.yaml` that looks like this:

```
apiVersion: clusterlink.net/v1alpha1
kind: Export
metadata:
  name: <service-name>
  namespace: <service-namespace>
spec:
  port: <port-number>
```

```
kubectl apply -f service-export.yaml
```
To import a service on a cluster, create a `service-import.yaml` that looks like this:

```
apiVersion: clusterlink.net/v1alpha1
kind: Import
metadata:
  name: <service-name>
  namespace: <service-namespace>
spec:
  port: <port-number>
  sources:
    - exportName:       <service-name>>
      exportNamespace:  <service-namespace>
      peer:             <peer-name>
```

```
kubectl apply -f service-import.yaml
```

Before performing Import/Export, it may be necessary to provide permissions to perform these operations.
This can be done by defining the proper `ClusterRole` and `ClusterRoleBinding`.

```
apiVersion: rbac.authorization.k8s.io/v1
kind: ClusterRole
metadata:
  name: cl-controlplane
rules:
- apiGroups: ["clusterlink.net"]
  resources: ["imports","exports", "peers", "accesspolicies", "privilegedaccesspolicies"]
  verbs: ["get", "list", "watch", "create", "delete", "update"] 
---
apiVersion: rbac.authorization.k8s.io/v1
kind: ClusterRoleBinding
metadata:
  name: cl-controlplane
roleRef:
  apiGroup: rbac.authorization.k8s.io
  kind: ClusterRole
  name: cl-controlplane
subjects:
- kind: ServiceAccount
  name: default
  namespace: clusterlink-system
```

### To detach cluster from ClusterLink fabric

```
clusterlink delete peer --name <peer_name>
```

