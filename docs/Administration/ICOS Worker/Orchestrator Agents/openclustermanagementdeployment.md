# Open Cluster Management Deployment

Open Cluster Management installation is installed in two phases:

1. [Control Plane Deployment](../../ICOS Agent/Orchestrators/controlplane.md): it provides a control plane on top of a Kubernetes node or cluster 
to become the controller of the Open Cluster Management environment
2. [Managed Clusters Aggregation](#managed-cluster-aggregation): it joins cluster/s to the previously installed control plane 
to start being orchestrated by the mentioned controller.



## Managed Cluster Aggregation:
As part of managed cluster aggregation process the following prerequisites must be satisfied:

* Ensure kubectl and kustomize are installed.
* The managed clusters should be <span style="color: red;">v1.11+</span> .

To join a cluster to the OCM Controller it is necessary to install clusteradm client on that cluster 
as for the controller itself during the previous section.
The steps are:

1. To Install clusteradm on the edge device

	```bash
	
	curl -L https://raw.githubusercontent.com/open-cluster-management-io/clusteradm/main/install.sh | bash
	
	```

	For easing the installation process, the following environment variables can be declared:
	
	```bash
	 # The context name of the clusters in your kubeconfig
	 export CTX_HUB_CLUSTER=<your hub cluster context>
	 export CTX_MANAGED_CLUSTER=<your managed cluster context>
	```
	
2. Ask to the ICOS support team a joining token for OCM.

3. Prepare to join the OCM Controller running on the ICOS Agent
	Finally, the following command must be executed on the cluster to become managed by OCM:
	
	```bash
	clusteradm join \
	    --hub-token <your token data> \
	    --hub-apiserver <your hub cluster endpoint> \
	    --wait \
	    --cluster-name "cluster1" \  # Or other arbitrary unique name
	    --context ${CTX_MANAGED_CLUSTER}
	```
	
    For example:
		
	```bash
	clusteradm join \
	    --hub-token <provided_token> \
	    --hub-apiserver https://10.160.3.240:6443 \
	    --wait \
	    --cluster-name "cluster1" \  # Or other arbitrary unique name

	```
	To obtain a valid token the below shown command must be executed on the controller cluster:
	
	```bash
	clusteradm get token
	```
4. Wait for the request to be accepted. Now your cluster has joined the ICOS Agent.
   After the join command is executed, the join request is sent, and it needs to be accepted 
   from the OCM controller cluster. To do so the following steps must be followed:
	
	a. Wait for CSR object creation on the controller cluster:
	
	```bash
	kubectl get csr -w --context ${CTX_HUB_CLUSTER} | grep cluster1  # or the previously chosen cluster name
	
	#pending CSR request example: 
	
	cluster1-tqcjj   33s   kubernetes.io/kube-apiserver-client   system:serviceaccount:open-cluster-management:cluster-bootstrap   Pending
	```
	
	b.  Accept the mentioned CSR request:
	
	```sh
	clusteradm accept --clusters cluster1 --context ${CTX_HUB_CLUSTER}
	```
	
	c.  It should be verified that the agents are properly installed and running onto the managed cluster:
	
	```sh
	kubectl -n open-cluster-management-agent get pod --context ${CTX_MANAGED_CLUSTER}
	```

5. Verify that the agents are properly installed and running onto your cluster:
	
	```sh
	kubectl -n open-cluster-management-agent get pod
	
	
	|NAME       |HUB ACCEPTED|MANAGED CLUSTER URLS|JOINED|AVAILABLE| AGE  |
	|-----------|------------|--------------------|------|---------|------|
	|cluster1   |true        |<your endpoint>     |True  | True    | 5m23s|
	
	
	```

	The output should look something like this:

	|NAME                                          |READY|STATUS  |
	|----------------------------------------------|-----|--------|
	|klusterlet-registration-agent-598fd79988-jxx7n| 1/1 | Running|
	|klusterlet-work-agent-7d47f4b5c5-dnkqw        | 1/1 | Running|
	


