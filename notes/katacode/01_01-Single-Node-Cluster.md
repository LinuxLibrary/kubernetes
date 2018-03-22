# Kubernetes - Launch Single Node Kubernetes Cluster

# Start Minikube

- After installing and configuring Minikube on your machine check whether it is properly installed or not

	```
	# minikube version
	```

- Start the cluster

	```
	# minikube start
	```

# Cluster Info

- The cluster can be interacted with using the kubectl CLI. This is the main approach used for managing Kubernetes and the applications running on top of the cluster.
- Details of the cluster and it's health status can be discovered via `kubectl cluster-info`
- To view the nodes in the cluster using `kubectl get nodes`

# Deploy Containers

- With a running Kubernetes cluster, containers can now be deployed.
- Using `kubectl run`, it allows containers to be deployed onto the cluster

	```
	# kubectl run first-deployment --image=katacoda/docker-http-server --port=80
	```

- The status of the deployment can be discovered via the running Pods

	```
	# kubectl get pods
	```

- Once the container is running it can be exposed via different networking options, depending on requirements.
- One possible solution is NodePort, that provides a dynamic port to a container.

	```
	# kubectl expose deployment first-deployment --port=80 --type=NodePort
	```

- Find the allocated port and export it

	```
	# export PORT=$(kubectl get svc first-deployment | grep first-deployment | awk -F: {'print $2'} |awk -F/ '{print $1}')
	```

- Hit the host on that port and ensure that you are able to access the application

	```
	# curl <HOSTNAME>:$PORT
	```
