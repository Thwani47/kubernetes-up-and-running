- Service discovery tools help solve the problem of finding which processes are listening at which addresses for which services
- The Domain Name System (DNS) is the traditional system of service discovery for the internet.
- Real service discovery in Kubernetes starts with a `Service` object
- A Service object is a way to create a named label selector
- We can use `kubectl expose` to create a service
  ```bash
	  kubectl expose deployment alpaca-prod
	```
- The `kubernetes` service is created automatically for us so we can find and talk to the Kubernetes API from within the app
- A service is assigned a type of virtual IP address called a `clusterIP`
	- This is a special IP address the system will load-balance across all of the Pods that are identified by the selector
- The Service object tracks which of our Pods are ready performing a readiness check
