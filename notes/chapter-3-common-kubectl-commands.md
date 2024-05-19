## Namespaces
- Kubernetes uses namespaces to organize objects in the cluster
- We can think of each namespace as a folder that holds a set of objects
- By default, `kubectl` interacts with the `default` namespace
- If we want to use a different namespace, we can pass the `--namespace` flag
	- For example, `kubectl --namespace=my-namespace` to reference objects in the `my-namespace` namespace
- If we want to reference all objects in all namespaces, we can pass the `--all-namespaces` flag
	- For example, to list nodes in all namespaces we can run
	  ```bash
	  kubectl get nodes --all-namespaces
		```
## Contexts
- If we want to change the `default` namespace more permanently, we can use a `context`
- The context gets recorded in the kubectl config file
- This config file also stores how to both find and authenticate to our cluster
- We can create a context with a different default namespace for kubectl to use by running
  ```bash
  kubectl config set-context my-context --namespace=my-namespace
	```
- This creates a new context, but does not switch to that context to start using it. To switch to the context we can run
  ```bash
  kubectl config use-context my-context
	```
- Contexts can also be used to manage different clusters or users for authenticating to those clusters using the `--users` and `--clusters` flags
## Viewing Kubernetes API Objects
- Everything contained in Kubernetes is represented by a RESTful resource
	- Each Kubernetes objects exists at a unique HTTP path
- The `kubectl` commands make HTTP requests to these URLs to access the Kubernetes objects
- The most basic command to view Kubernetes objects is `kubectl get`
	- `kubectl get <resource-name>` will return a list of all resources in the current namespace
		- for example, `kubectl get nodes` lists all nodes in the current namespace
	- `kubectl get <source-name> <object-name>` will return details about a specific resource
		- for example, `kubectl get node docker-desktop` will return the details about the `docker-desktop` node
- We can add the `-o wide` flag to get more details from `kubectl`
- We can also have the `kubectl` output in JSON or YAML by using either the `-o json` or `-o yaml` flags respectively
- We can view different types of resources by a comma separated list of object types. For example, 
  ```bash
  kubectl get nodes,services
	```
	will print all nodes and services for current namespace
- To get more details about a particular object, we can use the `describe command`
  ```bash
  kubectl describe node docker-desktop
	```
- To get a list of supported fields for a Kubernetes object, we can use the `explain` command
  ```bash
  kubectl explain nodes
	```
- Sometimes we want to continuously watch the state of a particular resource and see changes as they occur. We can use the `--watch` flag for this
  ```bash
  kubectl get node docker-desktop --watch
	```
## Creating, Updating, and Destroying Kubernetes Objects
- Objects in Kubernetes are represented as JSON or YAML files. 
- These files are either returned by the Kubernetes API as a response to a request (such as `kubectl get`) or posted to the API as part of a request
- We can use the JSON or YAML files to create, update, or delete objects in Kubernetes
- We use the `apply` command to create or modify Kubernetes objects
  ```bash
  kubectl apply -f obj.yaml
	```
- We can run this command a multiple times and Kubernetes will only make changes if there's any change in the file
- We can use the `--dry-run` flag to see what the `apply` command will do without actually performing the changes.
- We can use the `edit` command to interactively make changes to an object. 
  ```bash
	  kubectl edit <resource-name> <object-name>
	  ```
	*This will download the object's latest state, and launch an editor that contains the object's definition. After you save the changes, the changes are deployed to Kubernetes*
- The `apply` command also records the history of previous configurations. We can use the `edit-last-applied`, `set-last-applied`, and `view-last-applied` commands
- To delete an object we run
  ```bash
	  kubectl delete -f obj.yaml
	```
- `kubectl` will not prompt you to confirm the deletion
- We can also delete the object using its name
  ```bash
	  kubectl delete <resource-name> <object-name>
	```
## Labeling and Annotating Objects
- Labels and annotations are tags for your objects
- We can update the labels and annotations on any Kubernetes object using the `label` and `annotate` commands respectively. For example
  ```bash
  kubectl label pods bar color=red
	```
	This gives the pod `bar` a label `color` with the value of `red`. The syntax for annotations is the same
- `label` and `annotate` will not allow you to overwrite an existing label or annotation. We need to use the `--overwrite` flag to do that
- To remove a label we use the `<label-name>-` syntax. For example, to remove the label we set above we use
  ```bash
	  kubectl label pods bar color-
	```
## Debugging Commands
- To view the logs for a running container we can use the `logs` command
  ```bash
	  kubectl logs <pod-name>
	```
	- If you have multiple containers running in your pod you can specify the container by using the `-c` flag
	- By default, `kubectl logs` lists the current logs and exits. To stream the logs continuously, we use the `-f` flag  (which stands for follow)
- We can use `exec` to execute commands in a running container
  ```bash
	  kubectl exec -it <pod-name> -- bash
	```
	This will provide an interactive shell inside the running container
- We can also use `attach` to achieve the same thing
  ```bash
	  kubectl attach -it <pod-name>
	```
- We can use the `cp` command to copy files to and from our machines to a container
  ```bash
  kubectl cp <pod-name>:</path/to/store/the/file> </path/to/local/file>
  kubectl cp </path/to/local/file> <pod-name>:</path/to/store/the/file>
	```
	The command uses `kubectl cp <source> <destination>` format
- To access a Pod via the network, we can use the `port-forward`  command to forward network traffic from the local machine to the Pod. For example
  ```bash
	  kubectl port-forward <pod-name> 8080:80
	```
	This directs network from port `8080` on the local machine to the remote container on port `80`
- We can use `kubectl get events` to see a list of the latest 10 events on all objects on given namespace
	- We can add the `--watch` flag to stream events as they occur
- We can use the `top` command to see how our cluster objects are using our resources. This returns stats such as CPU usage and memory usage
  ```bash
	  kubectl top <resource-name>
	```
	By default, `top` only shows resources in the current namespace. We can the `--all-namespaces` to view all resources
## Cluster Management
- We can use `kubectl` to manage the cluster itself
- We can use the `cordon` and `drain` commands.
	- The `cordon` command prevents any future Pods to be scheduled on a particular node
	- The `drain` commands removes the node from the cluster
- This might be useful in a multi-node cluster where we have a node that needs to go for repairs. We can use `cordon` to prevent any Pods to be scheduled on said node, and then use `drain` to safely remove the node from the cluster.
- Once the machine is repaired, we can use `undrain` to add the node back to the cluster, and `uncordon` to re-enable Pod scheduling on the node
- These commands are useful if the machine needs to be unavailable for a long period
