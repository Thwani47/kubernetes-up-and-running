- Not really much notes to go through here. The chapter is showing how to install K8s using different approaches:
	- Cloud setup:
		- Azure Kubernetes setup
		- Google Cloud Kubernetes setup
		- AWS Kubernetes setup
	- Local machine setup:
		- Minikube
		- Kind (Kubernetes in Docker)
	- Kubernetes on Raspberry Pi
- The official Kubernetes client is the `kubectl`, a CLI tool used to interact with the Kubernetes API. 
- `kubectl` can be used to manage most Kubernetes services such as Pods, ReplicaSets, and Services.
- `kubectl` can also be used to explore and verify the health of the Kubernetes cluster
- To check the version of the Kubernetes cluster you can run
	```bash
    kubectl version
	```
- To get a diagnostics of the cluster we can run
```bash
kubectl get componentstatuses
```
- To list the worker nodes in your cluster you can run
```bash
kubectl get nodes
```
*This will return a single node if you're running a single-node K8s cluster using either Minikube or Docker Desktop* 
- You can use `kubectl describe` to get more information about  a specific node
```bash
kubectl describe node <node-name>
```

## Cluster Components
- Many of the Kubernetes components that make up the Kubernetes cluster are actually deployed using Kubernetes itself
- All of these components run in the `kube-system` namespace
	- A `namespace` in Kubernetes is an entity for organizing Kubernetes resources. Think of it like a folder in a filesystem (or an Azure resource group)
- **Kubernetes Proxy** --> The Kubernetes proxy is responsible for routing network traffic to load-balanced services in the cluster
	- To do this, the proxy must be present on every node in the cluster
	- Kubernetes has an API object called a `DaemonSet`, which we will cover later, which is used in many clusters to achieve this. If your cluster runs the Kubernetes proxy with a DaemonSet, you can see the proxies by running
	  ```bash
	  kubectl get daemonsets --namespace=kube-system kube-proxy
		```
- **Kubernetes DNS** --> Kubernetes also runs a DNS server, which provides naming and discovery for the services running in the cluster
	- Depending on the size of your cluster, you may have one or more DNS servers running in your cluster
	- The DNS service is run as a Kubernetes deployment, which manages these replicas.
	- The DNS service may be named `coredns` or some variant.
	- You can see the DNS servers running on your cluster by running
	  ```bash
	  kubectl get deployments --namespace=kube-system coredns
		```
	 - There is also a Kubernetes service that performs load balancing for the DNS server.
	 - This service may be named `core-dns` or `kube-dns` or some variant
	 - You can view that by running
	   ```bash
	   kubectl get services --namespace=kube-system kube-dns
		```
	- **Kubernetes UI** --> If we want to visualize our cluster, we can install the Kubernetes Web UI Dashboard
		- We can do that by following the instructions on [this page](https://kubernetes.io/docs/tasks/access-application-cluster/web-ui-dashboard/) if our cluster does not already have the dashboard provided
	