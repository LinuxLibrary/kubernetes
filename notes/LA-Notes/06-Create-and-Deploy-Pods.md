# Create and Deploy Pods

- For creating pods let us create a yaml template

	```
	mkdir Builds && cd $_
	vi nginx.yaml
	```
	
	```
	apiVersion: v1
	kind: Pod
	metadata: 
	  name: nginx
	spec:
	  containers:
	  - name: nginx
	    image: nginx:1.7.9
	    ports: 
	    - containerPort: 80
	```

- Let us create a pod based on our yaml file

	```
	kubectl create -f ./nginx.yaml
	```

- From the above you can launch a pod, but the traffic can't be routed to that pod from the VMs to the pods.
- For that we have to do port forward

	```
	kubectl port-forward nginx :80 &
	```

	- This will select a randon port from the local machine and route the traffic coming on to that port to the specified port of the pod (In above case it is 80)