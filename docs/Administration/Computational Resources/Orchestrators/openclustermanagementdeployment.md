# Open Cluster Management Deployment

Open Cluster Management installation is installed in two phases:

1. **Control Plane Deployment**: it provides a control plane on top of a Kubernetes node or cluster 
to become the controller of the Open Cluster Management environment
2. **Managed Clusters Aggregation**: it joins cluster/s to the previously installed control plane 
to start being orchestrated by the mentioned controller.

## Control Plane Deployment
As part of Open Cluster Management Controller installation, the following prerequisites must be 
satisfied:

* Ensure kubectl and kustomize are installed.
* The controller cluster should be Kubernetes <span style="color: red;">v1.19+</span>. 
(To run on controller cluster version **between [v1.16, v1.18]**, 
please manually enable feature gate ***V1beta1CSRAPICompatibility***).

* Currently, the bootstrap process relies on client authentication via CSR. 
Therefore, Kubernetes distributions that don't support it can't be used as the controller. 

For example: EKS.

To onboard OCM Controller it is necessary to install clusteradm client provided to correctly install 
all the components and dependencies required by the mentioned OCM.To do so, the described below command must be run:

```bash
curl -L https://raw.githubusercontent.com/open-cluster-management-io/clusteradm/main/install.sh | bash
```

For easing the installation process, the following environment variables should be declared:

```python
 # The context name of the clusters in your kubeconfig
 export CTX_HUB_CLUSTER=<your hub cluster context>
```

Finally to trigger the installation process the below command must be executed:

```bash
 # By default, it installs the latest release of the OCM components.
 # NOTE: For hub cluster version between v1.16 to v1.19 use the parameter: --use-bootstrap-token
 clusteradm init --wait --context ${CTX_HUB_CLUSTER}
```
Once installation is finished, it is possible to check that the tool is running properly:

```bash
kubectl -n open-cluster-management get pod --context ${CTX_HUB_CLUSTER}

|NAME                            |READY   |STATUS   |RESTARTS   |AGE|
|--------------------------------|--------|---------|-----------|---|
|cluster-manager-695d945d4d-5dn8k| 1/1    | Running | 0         |19d|

```

Additionally:

```bash
kubectl -n open-cluster-management-hub get pod --context ${CTX_HUB_CLUSTER}

|NAME                                                    |READY|STATUS |RESTARTS|AGE |
|--------------------------------------------------------|-----|-------|--------|----|
|cluster-manager-placement-controller-857f8f7654-x7sfz   | 1/1 |Running| 0      | 19d|
|cluster-manager-registration-controller-85b6bd784f-jbg8s| 1/1 |Running| 0      | 19d|
|cluster-manager-registration-webhook-59c9b89499-n7m2x   | 1/1 |Running| 0      | 19d|
|cluster-manager-work-webhook-59cf7dc855-shq5p           | 1/1 |Running| 0      | 19d|
|........................................................|.....|.......|........|....|
|........................................................|.....|.......|........|....|
```

## Managed Cluster Aggregation:
As part of managed cluster aggregation process the following prerequisites must be satisfied:

* Ensure kubectl and kustomize are installed.
* The managed clusters should be <span style="color: red;">v1.11+</span> .

To join a cluster to the OCM Controller it is necessary to install clusteradm client on that cluster as for the controller itself during the previous section.

To do so described below command must be executed:

```bash
curl -L https://raw.githubusercontent.com/open-cluster-management-io/clusteradm/main/install.sh | bash
```

For easing the installation process, the following environment variables can be declared:

```bash
 # The context name of the clusters in your kubeconfig
 export CTX_HUB_CLUSTER=<your hub cluster context>
 export CTX_MANAGED_CLUSTER=<your managed cluster context>
```

Finally, the following command must be executed on the cluster to become managed by OCM:

```bash
clusteradm join \
    --hub-token <your token data> \
    --hub-apiserver <your hub cluster endpoint> \
    --wait \
    --cluster-name "cluster1" \  # Or other arbitrary unique name
    --context ${CTX_MANAGED_CLUSTER}
```

To obtain a valid token the below shown command must be executed on the controller cluster:

```bash
clusteradm get token
```

After the join command is executed, the join request is sent, and it needs to be accepted 
from the OCM controller cluster. To do so the following steps must be followed:

**1.**  Wait for CSR object creation on the controller cluster:

```bash
kubectl get csr -w --context ${CTX_HUB_CLUSTER} | grep cluster1  # or the previously chosen cluster name

#pending CSR request example: 

cluster1-tqcjj   33s   kubernetes.io/kube-apiserver-client   system:serviceaccount:open-cluster-management:cluster-bootstrap   Pending
```

**2.**  Accept the mentioned CSR request:

```bash
clusteradm accept --clusters cluster1 --context ${CTX_HUB_CLUSTER}
```

**3.**  It should be verified that the agents are properly installed and running onto the managed cluster:

```bash
kubectl -n open-cluster-management-agent get pod --context ${CTX_MANAGED_CLUSTER}

|NAME                                          |READY|STATUS  |RESTARTS|AGE |
|----------------------------------------------|-----|--------|--------|----|
|klusterlet-registration-agent-598fd79988-jxx7n| 1/1 | Running| 0      | 19d|
|klusterlet-work-agent-7d47f4b5c5-dnkqw        | 1/1 | Running| 0      | 19d|
```

**4.**  The following command must be executed to list joined clusters on the controller:

```bash
kubectl get managedcluster --context ${CTX_HUB_CLUSTER}

#result example:

|NAME       |HUB ACCEPTED|MANAGED CLUSTER URLS|JOINED|AVAILABLE| AGE  |
|-----------|------------|--------------------|------|---------|------|
|cluster1   |true        |<your endpoint>     |True  | True    | 5m23s|
```