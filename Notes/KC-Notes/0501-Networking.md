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
- More details on the service configuration and active endpoints (Pods) can be viewed via 

	```
	# kubectl describe svc/webapp1-clusterip-svc
	```

- After deploying, the service can be accessed via the ClusterIP allocated.

	```
	# export CLUSTER_IP=$(kubectl get services/webapp1-clusterip-svc | grep IP: | awk '{print $2}')
	# echo CLUSTER_IP=$CLUSTER_IP
	# curl $CLUSTER_IP:80
	```

- Multiple requests will showcase how the service load balancers across multiple Pods based on the common label selector.

# Target Port

- Target ports allows us to separate the port the service is available on from the port the application is listening on.
- TargetPort is the Port which the application is configured to listen on.
- Port is how the application will be accessed from the outside.
- Similar to previously, the service and extra pods are deployed via 

	```
	# kubectl apply -f clusterip-target.yaml
	```

- The following commands will create the service.

	```
	# kubectl get svc
	# kubectl describe svc/webapp1-clusterip-targetport-svc
	```

- Definition for the clusterip-target and service can be found below

```
apiVersion: v1
kind: Service
metadata:
  name: webapp1-clusterip-targetport-svc  labels:    app: webapp1-clusterip-targetport
spec:
  ports:
  - port: 8080
    targetPort: 80
  selector:
    app: webapp1-clusterip-targetport
---
apiVersion: extensions/v1beta1
kind: Deploymentmetadata:
  name: webapp1-clusterip-targetport-deployment
spec:
  replicas: 2
  template:
    metadata:
      labels:
        app: webapp1-clusterip-targetport
    spec:
      containers:
      - name: webapp1-clusterip-targetport-pod
        image: katacoda/docker-http-server:latest
        ports:
        - containerPort: 80
---
```

- After the service and pods have deployed, it can be accessed via the cluster IP as before, but this time on the defined port 8080.

        ```
        # export CLUSTER_IP=$(kubectl get services/webapp1-clusterip-svc | grep IP: | awk '{print $2}')
        # echo CLUSTER_IP=$CLUSTER_IP
        # curl $CLUSTER_IP:8080
        ```

- The application itself is still configured to listen on port 80. Kubernetes Service manages the translation between the two.


