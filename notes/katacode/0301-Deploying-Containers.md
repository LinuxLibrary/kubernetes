# Kubernetes - Start containers using Kubectl

- Kubectl to create and launch Deployments, Replication Controllers and expose them via Services without writing yaml definitions. This allows you to quickly launch containers onto the cluster.

# Launch Cluster

- To start we need to launch a Kubernetes cluster.
- Execute the command below to start the cluster components and download the Kubectl CLI.

	```
	# minikube start
	```

- Wait for the Node to become Ready by checking 

	```
	# kubectl get nodes
	```
