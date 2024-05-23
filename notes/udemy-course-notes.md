# Kubernetes Overview
- Covers container concepts. 
	- Differences between Docker and VMs
	- container orchestration tools
		- Docker Swarm
		- Kubernetes
		- Mesos
- Covers the Kubernetes architecture
# Kubernetes Concepts
## Kubernetes YAML files
- covers yaml basics
	- writing lists, dictionaries, lists of dictionaries
	- spacing
- A Kubernetes definition file always contains 4 root-level required fields
	- apiVersion --> the version of the Kubernetes API we're using to create the objects. The value depends on the Kubernetes object we're creating
		- Pod - `v1`
		- Service - `v1`
		- ReplicaSet - `apps/v1`
		- Deployment - `apps/v1`
	- kind - The type of object we're trying to create. 
		- Values include `Pod`, `Service`, `ReplicaSet`, `Deployment`
	- metadata - Data about the object
		- object's name, labels
	- spec - The specification. Provides additional information to Kubernetes pertaining to the object
		- spec is a dictionary
## Pods
- The containers are encapsulated into Kubernetes objects called Pods
- A Pod is the smallest object we can create in Kubernetes
- Kubernetes does not run containers directly
- Pods usually have a 1-1 relationship with containers
	- To scale up, we add additional Pods. We don't add more containers on a Pod
- There also multi-container Pods
	- They are usually not containers of the same application
	- They are usually helper containers that perform helper functions to our application container
- To create a Pod we can use the `kubectl run` command. For example, this creates an nginx Pod
  ```bash
	  kubectl run nginx --image=nginx
	```
	Creates a Pod named nginx that runs an nginx container
- We can also use a YAML definition file
  ```yaml:pod-definition.yaml
	  apiVersion: v1
	  kind: Pod
	  metadata:
		  name: nginx
		  labels:
			  app: nginx
			  type: front-end
	  spec:
		  containers:
		    - name: nginx
		      image: nginx
	```
## Replication Controllers, and ReplicaSets
- Controllers are the brains behind Kubernetes
- The `Replication controller` helps us run multiple pods on our Kubernetes cluster
- The Replication Controller ensures that the desired number of Pods is always running 
- The Replication Controller helps balance load across multiple nodes
- Replication Controller is the older technology that is being replaced by `ReplicaSets`
- They serve the same purpose but work differently
- We can define Replication Controller using YAML as follows
  ```yaml:replication-controller-definition.yaml
	  apiVersion: v1
	  kind: ReplicationController
	  metadata:
	    name: nginx-replication-controller
	    labels:
	      app: nginx
	      type: front-end
	  spec:
	    replicas: 2
	    template:
	      metadata:
	        name: nginx
	        labels:
	          app: nginx
	          type: front-end
	      spec:
	        containers:
	        - name: nginx
	          image: nginx
	```
	We add the Pod definition in the `template` section of the Replication Controller since we want to replicate the `nginx` Pod
- We can define a `ReplicaSet` using YAML as follows
  ```yaml:replicaset-manifest.yaml
		apiVersion: apps/v1
		kind: ReplicaSet
		metadata:
		  name: nginx-replicaset
		  labels:
		    app: myapp
		    type: front-end
		spec:
		  replicas: 2
		  selector:
		    matchLabels:
		      type: front-end
		  template:
		    metadata:
		      name: nginx
		      labels:
		        app: nginx
		        type: front-end
		     spec:
		       containers:
		       - name: nginx
		         image: nginx
	```
	the `selector` helps the ReplicaSet identify the Pods that fall under it. The ReplicaSet is also able to manage pods that were created outside the ReplicaSet
- The `selector` is not a required field for the replication controller, but it is required for the ReplicaSet
	- The `selector` uses Pod labels to manage the Pods under it
- The ReplicaSet monitors the Pods and if a Pod fails, it is replaced and a new one is created
- If we wanted to scale a ReplicaSet, we can use either the `kubectl replace` or `kubectl scale` commands. For example, say we wanted to scale the ReplicaSet to run 6 Pods. We can 
	- Update the ReplicaSet definition file and set `spec.replicas` to `6` and then run `kubectl replace -f nginx-replicaset.yaml`
	- Update the ReplicaSet definition file and set `spec.replicas` to `6` and then run `kubectl scale --replicas=6 -f nginx-replicaset.yaml`
		- We can alternatively specify the ReplicaSet name instead of the manifest file as follows `kubectl scale --replicas=6 replicaset nginx-replicaset`
		- These do not update the number of replicas in the ReplicaSet definition file
## Deployments
- `Deployment` is a K8s object that allows us to upgrade our app instances, using rolling updates, pause updates, and roll back updates
- We define a Deployment as follows:
  ```yaml:nginx-deployment.yaml
	  apiVersion: apps/v1
	  kind: Deployment
	  metadata:
	    name: nginx-deployment
	    labels:
		  app: nginx
		  type: frontend
	  spec:
	    template:
	      metadata:
	        name: nginx
	        labels:
	          app: nginx
	          type: frontend
	       spec:
		     containers:
		     - name: nginx
		       image: nginx
		replicas: 3
		selector:
		  matchLabels:
		    type: frontend
	```
	The deployment automatically creates a `ReplicaSet`
- When we first create a Deployment, it triggers a `Rollout`. The rollout creates a Revision
	- These help us keep track changes made to our deployment and help us revert any changes if need be
	- We Can check the Rollout status using the `rollout status` command
	  ```bash
		  kubectl rollout status deployment/<deployment-name>
		```
	- We can checkout the history of a rollout using the `rollout history` command:
	  ```bash
		kubectl rollout history deployment/<deployment-name>
		```
- We have two types of deployment strategies:
	- **Recreate** - Destroy all running instances and create new instances
		- This causes an issue during the period when we've destroyed all running instances and are waiting for new instances to be spun up as our users may not be able to access our application
	- **Rolling Update** - The default strategy, this destroys one instance and creates its replacement while the other instances are running
- When we run `kubectl apply` to a deployment, a new rollout is triggered and a new revision is created
- We can also run `kubectl set image deployment/<deployment-name> <pod-name>=<image-name-to-be-set>` to update the image used in our deployment
	- i.e `kubectl set image deployment/nginx-deployment nginx=nginx:1.9.1`
		- This updates the image of the Pods (with the name `nginx`) in the `nginx-deployment` Deployment to use the `nginx:1.9.1` image
- We undo a change using the `rollout undo` command
  ```bash
	  kubectl rollout undo deployment/<deployment-name>
	```
- Kubernetes Deployment commands cheatsheet:

| Command   | Description                                             | Examples                                                                                                                         |
| --------- | ------------------------------------------------------- | -------------------------------------------------------------------------------------------------------------------------------- |
| `create`  | Create a deployment                                     | `kubectl create -f <deployment-definition.yaml>`                                                                                 |
| `get`     | List created deployments                                | `kubectl get deployments`<br><br>also, `kubectl get deploy`                                                                      |
| update    | Update an existing deployment                           | `kubectl apply -f <deployment-definition.yaml>`<br><br>`kubectl set image deloyment/<deployment-name> <pod name>:<image to use>` |
| `status`  | Check the status of rollouts                            | `kubectl rollout status deployment/<deployment-name>`                                                                            |
| `history` | Check the history of rollouts                           | `kubectl rollout history deployment/<deployment-name>`                                                                           |
| `undo`    | Rollback (undo the latest changes made to a deployment) | `kubectl rollout undo deployment/<deployment-name>`                                                                              |
- Add the `--record` flag to record actions performed against a Deployment to have them recorded and shown when running `kubectl rollout status` and `kubectl rollout history`
## Networking basics
- In Kubernetes, each Pod gets its own internal IP address
- A Kubernetes cluster has its own network with an address range
	- The Pods are assigned IP addresses that are within this range
## Services
- Kubernetes Services enable communication between various components from within and outside the cluster
- There are different types of Services
	- `NodePort Service` - This Service makes an internal port on the Pod accessible on the node running the Pod
		- We can access the Pod by through the IP address of the node and the exposed port
		- Assume we have a container running and accepting requests on port `80`
			- The `NodePort Service` will define `port 80` as the `targetPort` as this is the port it will direct traffic to
			- The Service will also have its own port, which is referred to as `port`. The Service is like a virtual server within the node. It has its own IP address and port
			- We also have the port on the node itself that we use to access the node externally, which is referred to as the `nodePort`. The node port has to be within the range `30000 - 32767`
			- So if we create a `NodePort` Service with `targetPort 80`, `port 3000` and `nodePort 30008`, clients outside the cluster will call `<node IP address>:30008` and the node will forward the request to the Service running at `<service IP address>:3000` which will forward the request to the Pod running at `<Pod IP adddress>:80`
		- We can create a `NodePort Service` as follows:
		  ```yaml:nodePort-service.yaml
			  apiVersion: v1
			  kind: Service
			  metadata:
			    name: myapp-service
			  spec:
				type: NodePort
				ports:
				- targetPort: 80
				  port: 3000
				  nodePort: 30008
				selector:
				  app: myapp
			```
			If we don't assign the `targetPort`, it is assumed to be the same as the `port`, and if we don't assign the `nodePort`, a random port within the valid range is assigned
		- Run `kubectl create -f service-definition.yaml` to create the service
		- Run `kubectl get services` to view services
		- The NodePort Service spans across multiple nodes in a multi-node cluster
	- `ClusterIP` - This creates an internal IP used by Pods to communicate with each other
		- We use the `ClusterIP Service` for internal communications of Pods within a Kubernetes cluster
		- The Service spans across the Pods assigned to it
		- The Service gets its own IP address and name, and the name is what its used to communicate with it
		- We can define a `ClusterIP Service` as follows
		  ```yaml:clusterip-service.yaml
			  apiVersion: v1
			  kind: Service
			  metadata:
				  name: myapp-service
			  spec:
				  type: ClusterIP
				  ports:
				    - targetPort: 80
				      port: 80
				  selector:
					  app: myapp
		  ```
		  - The `ClusterIP` Service type is the default Service type. If we do not specify `spec.type`, Kubernetes will create a `ClusterIP` Service
		  - The `ClusterIP` Service definition requires 2 ports:
			  - `targetPort` - This is the port exposed by the Pod
			  - `port` - This is the port exposed by the Service
		- Any client wanting to communicate with a Pod through the Service will send the requests to `<service-name>:<port>` and the Service will direct the traffic to `<pod IP address>:<targetPort>`
	- `LoadBalancer` - Creates a load balancer that can distribute load between Pods
		- The `LoadBalancer` service is an extension of the `NodePort` Service
		- We can create a `LoadBalancer Service` as follows:
		  ```yaml:loadbalancer-service.yaml
			  apiVersion: v1
			  kind: Service
			  metadata:
				  name: myapp-service
			  spec:
				  type: LoadBalancer
				  ports:
				    - targetPort: 80
				      port: 80
				      nodePort: 30008
				  selector:
					  app: myapp
		  ```

