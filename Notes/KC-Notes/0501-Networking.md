# Kubernetes - Networking Introduction

- Kubernetes have advanced networking capabilities that allow Pods and Services to communicate inside the cluster's network and externally.
- In this scenario, you will learn the following types of Kubernetes services.
	- Cluster IP
	- Target Ports
	- NodePort
	- External IPs
	- Load Balancer
- Kubernetes Services are an abstract that defines a policy and approach on how to access a set of Pods.
- The set of Pods accessed via a Service is based on a Label Selector.

# Cluster IP

- Cluster IP is the default approach when creating a Kubernetes Service.
- The service is allocated an internal IP that other components can use to access the pods.
- By having a single IP address it enables the service to be load balanced across multiple Pods.
- Services are deployed via 
	
	```
	# kubectl apply -f clusterip.yaml
	```

- Below is the definition

```
apiVersion: v1
kind: Service
metadata:
  name: webapp1-clusterip-svc
  labels:
    app: webapp1-clusterip
spec:
  ports:
  - port: 80
  selector:
    app: webapp1-clusterip
---
apiVersion: extensions/v1beta1
kind: Deployment
metadata:
  name: webapp1-clusterip-deployment
spec:
  replicas: 2
  template:
    metadata:
      labels:
        app: webapp1-clusterip
    spec:
      containers:
      - name: webapp1-clusterip-pod
        image: katacoda/docker-http-server:latest
        ports:
        - containerPort: 80
---
```

- This will deploy a web app with two replicas to showcase load balancing along with a service.
- The Pods can be viewed at:

	```
	# kubectl get pods
	```

- The same will create a service as well. To view it:

	```
	# kubectl get svc
	```
