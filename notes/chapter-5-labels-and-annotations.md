- Labels and annotations allow us to work in sets of things that map how we think about our applications
- Labels are key-value pairs that can be attached to Kubernetes objects such as Pods and ReplicaSets
	- They provide the foundation for grouping objects
	- They provide identifying metadata for objects
		- These are fundamental qualities of objects that will be used for grouping, viewing, and operating
	- Both the key and values are represented by strings
	- We can add labels as follows:
	  ```bash
		  $ kubectl run alpaca-prod --image=gcr.io/kuar-demo/kuard-amd64:blue --labels="ver=1,app=alpaca,env=prod"
		  $ kubectl run alpaca-test --image=gcr.io/kuar-demo/kuard-amd64:green --labels="ver=2,app=alpaca,env=test"
		```
	- We can apply or update labels to created objects as follows
	  ```bash
	  $ kubectl label pods alpaca-test "canary=true"
		```
	- We can remove a label from an object using the `<label-name>-` syntax. For example
	  ```bash
	  kubectl label pods alpaca-test "canary-"
		```
	- We can use `label selectors` to filter Kubernetes objects based on a set of labels. For example, to see on `version 1` apps we can run
	  ```bash
	  kubectl get pods --selector="ver=1"
		```
		Specifying more than 1 label, Kubernetes uses local AND operation to match the objects
	- We can also use the `in` syntax to search for apps that labels in a list. For example
	  ```bash
		  kubectl get pods --selector="ver in (1, 2)"
		```
	- We have these operators for the label selectors
	  

| Operator                    | Description                                                                                                    |
| --------------------------- | -------------------------------------------------------------------------------------------------------------- |
| `key=value`                 | Matches objects with `key` set to `value`                                                                      |
| `key!=value`                | Matches all objects with `key` not set to `value`                                                              |
| `key in (value1, value2)`   | Matches all objects where `key` is one of `value1` or `value2`                                                 |
| `key notin (value1, value2` | Matches all objects where `key` is not one of `value1` or `value2`                                             |
| `key`                       | Matches all objects with `key` set to anything (`key` is present in the label list and is not an empty string) |
| `!key`                      | Matches all objects where`key` is not set. These objects do not have a label `key`                             |

- Annotations provide a storage mechanism that resembles labels.
	- They are key-value pairs that hold non-indentifying information that tools and libraries can use
	- Annotations are used to provide extra information about where an object came from, how to use it, or the policy around that object
	- Annotations are used to
		- Keep track of a reason for the latest update on an object
		- Communicate a specialized scheduling policy to a specialized schedular
		- Extend data about the last tool to update the resource and how it was updated
		- Attach build, release, or image information that isn't appropriate for labels (Git hash, timestamp, PR number, etc)
		- Provide extra data to enhance the visual quality of objects (we can use annotations to add links to icons to show in the UI)
	- Annotations are defined in the `metadata` section in the object's manifest
	  ```yaml
		  ...
		  metadata:
			  annotations:
				  example.com/icon-url: "https://example.com/icon.png"
		```
		