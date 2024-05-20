- In Kubernetes, each Pod gets its own internal IP address
	- But the Pods in Kubernetes are ephemeral (they are are destroyed frequently)
	- When a Pod gets replaced, it gets a new IP address
	- It does not make sense to use a Pod IP address because we'd have to cater for a new IP address every time a new Pod gets created
- A Service provides a static IP address that does not change even when a Pod dies
	- In front of each Pod we set a Service which represents a stable IP address to access a Pod
- A Service also provides load balancing
	- When we have Pod replicas, the Service will load balance the traffic across the replicas.
	- Clients call a single stable single IP address instead of calling each Pod individually
- Services are a good abstraction for loose coupling from within and outside the cluster
- There are several types of Services in Kubernetes:

## ClusterIP Service
- This is the default Service type that gets created when we don't specify the Service type
- Also called an `Internal Service`
- The Service will be accessible at a specific IP address and port
- A Service identifies the Pods it forwards the requests to using selectors (labels)
- When defining a Service, we use `targetPort` to specify the port to which requests will be forwarded to on the Pod
	- The selectors determine the Pods (service endpoints) and the `targetPort` identifies the Pod port
- When we create a Service, Kubernetes will create an `Endpoint` object that has the same name as the Service
	- This object helps Kubernetes keep track of which Pods are members of which Service
- The Service has two ports:
	- `port` (or Service port) --> The port through which the Service can be communicated with on (i.e, Ingress will forward traffic to the service on this port)
	- `targetPort` --> The port on the Pods that the Service will forward traffic to (this has to match the port the container on the Pod is listening on)
- We can also define a `Multi-Port Service` which exposes multiple ports
	- For example, assume we have a Pod running two containers, one container for the application running on port `3000`, and one container exporting the application logs to a log aggregation tool like Prometheus, running on port `4000`. To expose both these ports, we can define our Service as a Multi-port Service like
	  ```yaml
		  apiVersion: v1
		  kind: Service
		  metadata:
			  name: microservice-demo
		  spec:
			  selector:
				  app: my-microservice
			  ports:
				  - name: my-awesome-app
				    protocol: TCP
					port: 3000 # expose port 3000 on the Service
					targetPort: 3000 # the container is listening on port 3000 in the Pod
				  - name: my-log-exporter
					protocol: TCP
					port: 4000
					targetPort: 4000
		```
		When we have multiple ports, we have to specify the name field in the ports section.
## Headless Service
- We use the Headless Service when
	- The client wants to communicate with 1 specific Pod directly
	- Pods want to talk directly with a specific Pod
- This is useful when we're deploying Stateful applications, like databases
	- With Stateful applications, the Pod replicas are not identical, each one has it's own state
		> For example, we might have a `MySQL` database deployment with 3 replicas, 1 master DB and 2 DB replicas, we only want to write data to the master DB and have the replicas synchronize their data continuously, and when we add a new replica, it must connect to the latest updated replica and clone data from it.
		> 
		> This is one use case for communicating directly with Pods
		
- The client needs to know the IP addresses of each Pod
	- This can be done by either calling to the Kubernetes API
	- Or be done via a DNS lookup
		- This will return a single IP address that belongs to a Service (`clusterIP`)
		- We can instruct Kubernetes to not return the cluster IP address by setting the `clusterIP` field to `None`. This will return the Pod IP address instead
		  ```yaml
		  apiVersion: v1
		  kind: Service
		  metadata:
			  name: microservices-demo
		  spec:
			  clusterIP: None
			  selector: 
				  app: my-awesome-app
			  ports:
				  - protocol: TCP
				    port: 3000
				    targetPort: 3000
		```

- When we define a Service configuration, we can define the type of the Service
	- `ClusterIP`
		- Internal Service
		- The default. If no type is specified, it is set to `ClusterIP`
	- `NodePort`
	- `LoadBalancer`
## NodePort Service
- `NodePort` Service creates a Service that is accessible on a static port on each worker node. 
	- The `ClusterIP` Service is only accessible within the cluster
	- No external traffic can directly reach the `ClusterIP` Service
	- The `NodePort` Service allows external traffic to reach it on a fixed port on each worker node
		- We can make requests from the browser to the Node without having to configure Ingress
- When we define a NodePort Service, we need to specify the `nodePort` field in the ports section
	- The nodePort value has to be within the range `30000-32767`
- External traffic can be directed to `<worker-node-ip-address>:<node-port>`
- When we create a `nodePort` Service, a `ClusterIP` Service is automatically created
	- Therefore we also have to specify a `port` field for the `clusterIP` Service
	- The `ClusterIP` Service will span across all the worker nodes
- The NodePort Service is not entirely secure
## LoadBalancer Service
- The LoadBalancer makes the `ClusterIP` Service accessible through the cloud providers load balancer
- When we create a LoadBalancer Service, NodePort and ClusterIP Services are created automatically
	- We also need to specify the `port`, `nodePort`, and `targerPort`
		- The nodePort is only accessible through the load balancer
- It is an extension of the NodePort Service

		
		
		