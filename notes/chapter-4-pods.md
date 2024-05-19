## Pods in Kubernetes
Kubernetes groups multiple containers into a single atomic unit called a `Pod`
	- The name goes with the whale theme of Docker containers, since a group of whales is referred to as a Pod
- A Pod is a collection of application containers and volumes running in the same execution environment.
- Pods, not containers, are the smallest deployable artifact in Kubernetes
	- All containers on the same Pod always land on the same machine
- Applications running in the same Pod:
	- share the same IP address and port space,
	- have the same hostnames and
	- can communicate using native interprocess channels
- Containers on different Pods but on the same node might be as well be running on different servers
## Thinking with Pods
- What do we put in a Pod?
> Consider a application that has a `WordPress` site and a `MySQL` database. One approach would be to want to put the WordPress container and the MySQL container in the same Pod. 
> 
> As much as this is feasible, it is an anti-pattern of Pod construction. There are two reasons why this is anti-pattern.
> 
> 1. WordPress and the MySQL database are not truly symbiotic. The WordPress container and the database container can live on different machines and still be able to communicate with one another over the internet.
> 2. You don't want to scale the Database every time you scale the WordPress site. You may want to add more instances of the WordPress site to make it highly available to your customers. If you have it running in the same Pod as the Database, every replica will result in a new database replica being created. This will result in more resources being utilized, and frankly, we don't wanna do this.

- The question isn't what do we put in a Pod. The right question to ask rather is `Will these containers work correcly if they land on different machines?`.
	- If the answer is no, those containers need to be put in one Pod. 
	- If the answer is yes, then having multiple Pods is probably the correct answer
## The Pod Manifest
- Pods are described in a Pod manifest file, which is just a text-file representation of the Kubernetes API object
- The manifest files allow us to use declarative configuration, which allows us to describe the desired state we'd like our cluster in, and submit that desired state, and Kubernetes will take the relevant actions to bring our cluster to that desired state
- The Kubernetes API server accepts and validates Pod manifests before storing them in `etcd`
- The `kube-scheduler` uses the `kube-apisever` to find nodes that haven't been scheduled
## Creating a Pod
- The simplest way to create a Pod is via the `kubectl run` command.  For example, 
  ```bash
	  kubectl run kuard --image=gcr.io/kuar-demo/kuard-amd64:blue
	```
	You can see the status of the Pod by running `kubectl get pods`
- We can delete the pod by running `kubectl delete pod kuard`
## Creating a Pod Manifest
- Pod manifests include a couple of key fields and attributes, namely
	- a `metadata` section for describing the Pod and its labels
	- a `spec` section for describing the volumes, and a list of containers that will run in the Pod
- Consider this Docker command
  ```bash
		docker run -d --name kuard -p 8080:80 gcr.io/kuar-demo/kuard-amd64:blue  
	```
	This runs a `kuard-amd64` container with the name `kuard` and exposes the port `80` on the container to port `8080` on the host machine. 
- We can achieve a similar result by creating a Pod manifest as follows
  ```yaml:kuard-pod.yaml
	apiVersion: v1
	kind: Pod
	metadata:
		name: kuard
	spec:
		containers:
			- image: gcr.io/kuar-demo/kuard-amd64:blue
			  name: kuard
			  ports:
				  - containerPort: 8080
					name: http
					protocol: TCP
	```
- We can create a Pod by running `kubectl apply -f kuard-pod.yaml`
## Accessing your Pod
### Using Port Forwarding
- We can use the `port-foward` command to create a secure tunnel from our local machine, through Kubernetes, to the instance of the Pod running on one of the worker nodes
  `kubectl port-foward kuard 8080:8080`
  This allows us to access the Pod at [http://localhost:8080](http://localhost:8080) on our local machine.
### Getting more info with Logs
- We can run `kubectl logs kuard` to download the current logs from the running instance of `kuard`
	- We can add the `--previous` flag to the `logs` command to get the logs of the previous instance
### Running commands in your Container with exec
- In order to execute commands inside our container, we can run `kubectl exec kuard -- <command>`
	- `kubectl exec kuard -- date` executes the `date` command inside the container, an prints it to the terminal
	- We can get an interactive session by adding the `-it` flag
		- `kubectl exec -it kuard -- date`
### Copying files to and from Containers
- Suppose we had a file names `data.txt` we wanted to coy to our container. We would run `kubectl cp ./data.txt kuard:/data/data.txt `
## Health Checks
- Kubernetes uses the health checks of our containers to ensure they are always running. If the health check fails, Kubernetes restarts the container
	- The health check checks for the application's main process. 
- Sometimes the main process might be running (it'll be healthy as far as health checks are concerned), but the application might be stuck. 
	- Kubernetes introduced `liveness checks` that run against the apps logic, like loading a web page to verify that the app is not just running, but it is functioning properly
- Since the health checks and liveness checks are application-speciic, we have to define then in our Pod manifest
### Liveness Probe
- Liveness probes are defined per container, which means each container inside the Pod is health-checked separately
- We can add a liveness probe to the kuard Pod as follows
  ```yaml
	apiVersion: v1
	kind: Pod
	metadata:
		name: kuard
	spec:
		containers:
			- image: gcr.io/kuar-demo/kuard-amd64:blue
			  name: kuard
			  livenessProbe:
				  httpGet:
					  path: /healthy
					  port: 8080
				  initialDelaySeconds: 5
				  timeoutSeconds: 1
				  periodSeconds: 10
				  failureThreshold: 3
			  ports:
				  - containerPort: 8080
					name: http
					protocol: TCP
	```
	This adds an `httpGet`probe to perform an HTTP GET request to the `/healthy` endpoint on port `8080` on the `kuard` container.  The `initialDelaySeconds` instructs Kubernetes to not start the liveness probe until 5 seconds has elapsed since the containers in the Pod have been created. The `timeoutSeconds` instructs Kubernetes that the probe must respond with a timeout value of `1s` and that the response's HTTP Status code must be equal to or greater than `200` and less than `400`. If the `/healthy` endpoint does not respond within `1s`, it'll be considered unhealthy. `periodSeconds` instructs Kubernetes to call the probe every 10 seconds. The `failureThreshold` instructs Kubernetes to restart the container if the probe fails over 3 consecutive times
- The default response to a failed liveness check is to restart the Pod. We can specify the `restartPolicy` where we have 3 options:
	- `Always` (the default)
	- `OnFailure`
	- `Never`
### Readiness Probe
- Kubernetes makes a distinction between `liveness` and `readiness`
	- `Liveness` determines if an app is running properly. Containers that fail the liveness check are restarted
	- `Readiness` determines if an app is ready to serve user requests. Containers that fail readiness checks are removed from the service load balancers
- Readiness checks are configured similar to liveness checks
- We'll explore readiness check in later chapters
### Other types of health checks
- Kubernetes supports non-HTTP health checks. It supports
	- `tcpSocket` health check which attempts to open a TCP socket connection. If the connection succeeds, the probe succeeds. 
		- This is useful for applications like databases or non-HTTP based APIs
	- `exec probes` - These execute a script or a command in the context of the container. If the script returns a zero exit code, the probe succeeds, else it fails
## Resource Management
- Kubernetes allows us to specify two types of resource metrics:
	- `resource requests` - These specify the minimum amount of resource required to run the application
		- The most commonly requested resources are CPU and memory. Kubernetes also supports GPU
		- We add these requests on the container. For example, to add resource requests to the `kuard` Pod we can add
		  ```yaml
				apiVersion: v1
				kind: Pod
				metadata:
					name: kuard
				spec:
					containers:
						- image: gcr.io/kuar-demo/kuard-amd64:blue
						  name: kuard
						  resources:
							  requests:
								  cpu: "500m"
								  memory: "128Mi"
						  livenessProbe:
							  httpGet:
								  path: /healthy
								  port: 8080
							  initialDelaySeconds: 5
							  timeoutSeconds: 1
							  periodSeconds: 10
							  failureThreshold: 3
						  ports:
							  - containerPort: 8080
								name: http
								protocol: TCP
			```
		Resources are requested per container, and not per Pod. The total resources requested by a Pod is the sum of the resources requested by the Pod's containers
	- `resource limits` - These specify the maximum amount of resource an application can consume
		- The Kubernetes scheduler will ensure that the sum of all resource requests of a Pod does not exceed the capacity of the node
			- A  Pod is guaranteed to have access to at least the requested resources
		- We can add resource limits to the `kuard` container as follows
		  ```bash
			  apiVersion: v1
				kind: Pod
				metadata:
					name: kuard
				spec:
					containers:
						- image: gcr.io/kuar-demo/kuard-amd64:blue
						  name: kuard
						  resources:
							  requests:
								  cpu: "500m"
								  memory: "128Mi"
							  limits:
								  cpu: "1000m" 
								  memory: "256Mi"
						  livenessProbe:
							  httpGet:
								  path: /healthy
								  port: 8080
							  initialDelaySeconds: 5
							  timeoutSeconds: 1
							  periodSeconds: 10
							  failureThreshold: 3
						  ports:
							  - containerPort: 8080
								name: http
								protocol: TCP
			```
### Persisting Data with Volumes
- When a Pod is restarted or deleted, any data that was in the container's filesystem is lost
- We can persist container data using volumes
- There are two approaches to defining volumes on a Pod manifest
	- in the `spec.volumes` manifest section, which is an array of all the volumes that may be accessed by containers in the Pod
		- In the `volumeMounts` array in the container definition. 
			- Two different containers in a Pod can mount the same volume at different mount paths
- We can mount a volume to our `kuard` Pod as follows:
  ```yaml
		apiVersion: v1
		kind: Pod
		metadata:
			name: kuard
		spec:
			containers:
				- image: gcr.io/kuar-demo/kuard-amd64:blue
				  name: kuard
				  volumeMounts:
					  - mountPath: "/data"
						name: "kuard-data"
				  resources:
					  requests:
						  cpu: "500m"
						  memory: "128Mi"
					  limits:
						  cpu: "1000m" 
						  memory: "256Mi"
				  livenessProbe:
					  httpGet:
						  path: /healthy
						  port: 8080
					  initialDelaySeconds: 5
					  timeoutSeconds: 1
					  periodSeconds: 10
					  failureThreshold: 3
				  ports:
					  - containerPort: 8080
						name: http
						protocol: TCP

- The different ways we can use volumes in our applications include:
	- Communication/ Synchronization
	- Cache
	- Persistent data
	- Mounting the host filesystem