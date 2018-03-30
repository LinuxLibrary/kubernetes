# Kubernetes - Networking Introduction

- Kubernetes have advanced networking capabilities that allow Pods and Services to communicate inside the cluster's network and externally.
- In this scenario, you will learn the following types of Kubernetes services.
	- [Cluster IP](https://github.com/LinuxLibrary/kubernetes/blob/master/Notes/KC-Notes/0501-Networking.md#cluster-ip)
	- [Target Port](https://github.com/LinuxLibrary/kubernetes/blob/master/Notes/KC-Notes/0501-Networking.md#target-port)
	- [NodePort](https://github.com/LinuxLibrary/kubernetes/blob/master/Notes/KC-Notes/0501-Networking.md#nodeport)
	- [External IP](https://github.com/LinuxLibrary/kubernetes/blob/master/Notes/KC-Notes/0501-Networking.md#external-ip)
	- [Load Balancer](https://github.com/LinuxLibrary/kubernetes/blob/master/Notes/KC-Notes/0501-Networking.md#load-balancer)
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

# NodePort

- While TargetPort and ClusterIP make it available to inside the cluster, the NodePort exposes the service on each Node’s IP via the defined static port.
- No matter which Node within the cluster is accessed, the service will be reachable based on the port number defined.
- Let us apply the nodeport definition

	```
	# kubectl apply -f nodeport.yaml
	```

- Nodeport definition can be found below

```
apiVersion: v1
kind: Service
metadata:
  name: webapp1-nodeport-svc
  labels:
    app: webapp1-nodeport
spec:
  type: NodePort
  ports:
  - port: 80
    nodePort: 30080
  selector:
    app: webapp1-nodeport
---
apiVersion: extensions/v1beta1
kind: Deployment
metadata:
  name: webapp1-nodeport-deployment
spec:
  replicas: 2
  template:
    metadata:
      labels:
        app: webapp1-nodeport
    spec:
      containers:
      - name: webapp1-nodeport-pod
        image: katacoda/docker-http-server:latest
        ports:
        - containerPort: 80
---
```

- To view the details of the service

	```
	# kubectl get svc
	```

- To get detailed information of the service

	```
	# kubectl describe svc/webapp1-nodeport-svc
	```

- The service can now be reached via the Node's IP address on the NodePort defined.

# External IP

- Another approach to making a service available outside of the cluster is via External IP addresses.
- Update the definition to the current cluster's IP address with 

	```
	# sed -i 's/HOSTIP/172.17.0.30/g' externalip.yaml
	```

- Let us apply the definition

	```
	# kubectl apply -f externalip.yaml
	```

- The service is now bound to the IP address and Port 80 of the master node.

# Load Balancer

- When running in the cloud, such as EC2 or Azure, it's possible to configure and assign a Public IP address issued via the cloud provider.
- This will be issued via a Load Balancer such as ELB.
- This allows additional public IP addresses to be allocated to a Kubernetes cluster without interacting directly with the cloud provider.
- As Katacoda is not a cloud provider, it's still possible to dynamically allocate IP addresses to LoadBalancer type services.
- This is done by deploying the Cloud Provider using
	
	```
	# kubectl apply -f cloudprovider.yaml
	```
- When running in a service provided by a Cloud Provider this is not required.
- When a service requests a Load Balancer, the provider will allocate one from the 10.10.0.0/26 range defined in the configuration.

	```
	# kubectl get pods -n kube-system
	```

	```
	# kubectl apply -f loadbalancer.yaml
	```

- The service is configured via a Load Balancer as defined in

```
apiVersion: v1
kind: Service
metadata:
  name: webapp1-loadbalancer-svc
  labels:
    app: webapp1-loadbalancer
spec:
  type: LoadBalancer
  ports:
  - port: 80
  selector:
    app: webapp1-loadbalancer
---
apiVersion: extensions/v1beta1
kind: Deployment
metadata:
  name: webapp1-loadbalancer-deployment
spec:
  replicas: 2
  template:
    metadata:
      labels:
        app: webapp1-loadbalancer
    spec:
      containers:
      - name: webapp1-loadbalancer-pod
        image: katacoda/docker-http-server:latest
        ports:
        - containerPort: 80
---
```

- While the IP address is being defined, the service will show Pending. When allocated, it will appear in the service list.

	```
	# kubectl get svc
	# kubectl describe svc/webapp1-loadbalancer-svc
	```

- The service can now be accessed via the IP address assigned, in this case from the 10.10.0.0/26 range.

	```
	# export LoadBalancerIP=$(kubectl describe svc/webapp1-loadbalancer-svc | grep 'LoadBalancer Ingress:' | awk '{print $3}')
	# echo LoadBalancerIP=$LoadBalancerIP
	# curl $LoadBalancerIP
	```
