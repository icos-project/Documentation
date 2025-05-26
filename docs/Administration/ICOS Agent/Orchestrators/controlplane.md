# OCM - Control Plane Deployment

The **Control Plane Deployment** provides a control plane on top of a Kubernetes node or cluster 
to become the controller of the Open Cluster Management environment.

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
