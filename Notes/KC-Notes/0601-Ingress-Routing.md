# Kubernetes - Create Ingress Routing

- Kubernetes have advanced networking capabilities that allow Pods and Services to communicate inside the cluster's network.
- An Ingress enables inbound connections to the cluster, allowing external traffic to reach the correct Pod.
- Ingress enables externally-reachable urls, load balance traffic, terminate SSL, offer name based virtual hosting for a Kubernetes cluster.

# Create Deployment

- To start, deploy an example HTTP server that will be the target of our requests.
- The deployment contains three deployments, one called webapp1 and a second called webapp2, and a third called webapp3 with a service for each.
- Definition used for this can be found below

```
apiVersion: extensions/v1beta1
kind: Deployment
metadata:
  name: webapp1
spec:
  replicas: 1
  template:
    metadata:
      labels:
        app: webapp1
    spec:
      containers:
      - name: webapp1
        image: katacoda/docker-http-server:latest
        ports:
        - containerPort: 80
---
apiVersion: extensions/v1beta1
kind: Deployment
metadata:
  name: webapp2
spec:
  replicas: 1
  template:
    metadata:
      labels:
        app: webapp2
    spec:
      containers:
      - name: webapp2
        image: katacoda/docker-http-server:latest
        ports:
        - containerPort: 80
---
apiVersion: extensions/v1beta1
kind: Deployment
metadata:
  name: webapp3
spec:
  replicas: 1
  template:
    metadata:
      labels:
        app: webapp3
    spec:
      containers:
      - name: webapp3
        image: katacoda/docker-http-server:latest
        ports:
        - containerPort: 80
---
apiVersion: v1
kind: Service
metadata:
  name: webapp1-svc
  labels:
    app: webapp1
spec:
  ports:
  - port: 80
  selector:
    app: webapp1
---
apiVersion: v1
kind: Service
metadata:
  name: webapp2-svc
  labels:
    app: webapp2
spec:
  ports:
  - port: 80
  selector:
    app: webapp2
---
apiVersion: v1
kind: Service
metadata:
  name: webapp3-svc
  labels:
    app: webapp3
spec:
  ports:
  - port: 80
  selector:
    app: webapp3
---
```

- Let us deploy this defenition

	```
	# kubectl apply -f deployment.yaml
	```

- To view the status of the deployment

	```
	# kubectl get deployment
	```

# Deploy Ingress

- Ingress is deployed as a Replication Controller.
- This controller combines a software load balancer, such as Nginx or HAProxy, with Kubernetes integration for configuring itself based on the defined rules.
- The YAML file below defines a nginx-based Ingress controller together with a service making it available on Port 80 to external connections using ExternalIPs.
- If the Kubernetes cluster was running on a cloud provider then it would use a LoadBalancer service type.
- Definition for ingress can be found below

```
apiVersion: v1
kind: ReplicationController
metadata:
  name: nginx-ingress-rc
  labels:
    app: nginx-ingress
spec:
  replicas: 1
  selector:
    app: nginx-ingress
  template:
    metadata:
      labels:
        app: nginx-ingress
    spec:
      containers:
      - image: nginxdemos/nginx-ingress:0.9.0
        name: nginx-ingress
        ports:
        - containerPort: 80
          hostPort: 80
---
apiVersion: v1
kind: Service
metadata:
  name: nginx-ingress-lb
  labels:
    app: nginx-ingress
spec:
  externalIPs:
  - 172.17.0.4
  ports:
  - port: 80
    name: http
    targetPort: 80
  selector:
    app: nginx-ingress
---
```

- The Ingress controllers are deployed in a familiar fashion to other Kubernetes objects with 

	```
	# kubectl create -f ingress.yaml
	```

- The status can be identified using

	```
	# kubectl get rc
	```

# Deploy Ingress Rules

- Ingress rules are an object type with Kubernetes.
- The rules can be based on a request host (domain), or the path of the request, or a combination of both.
- An example set of rules are defined within below definition

```
apiVersion: extensions/v1beta1
kind: Ingress
metadata:
  name: webapp-ingress
spec:
  rules:
  - host: my.kubernetes.example
    http:
      paths:
      - path: /webapp1
        backend:
          serviceName: webapp1-svc
          servicePort: 80
      - path: /webapp2
        backend:
          serviceName: webapp2-svc
          servicePort: 80
      - backend:
          serviceName: webapp3-svc
          servicePort: 80
```

- The important parts of the rules are defined below.
- The rules apply to requests for the host my.kubernetes.example.
- Two rules are defined based on the path request with a single catch all definition.
- Requests to the path /webapp1 are forwarded onto the service webapp1-svc.
- Likewise, the requests to /webapp2 are forwarded to webapp2-svc.
- If no rules apply, webapp3-svc will be used.
- This demonstrates how an application's URL structure can behave independently about how the applications are deployed.
- As with all Kubernetes objects, they can be deployed via 

	```
	# kubectl create -f ingress-rules.yaml
	```

- Once deployed, the status of all the Ingress rules can be discovered via

	```
	# kubectl get ing
	```

- With the Ingress rules applied, the traffic will be routed to the defined place.
- The first request will be processed by the webapp1 deployment.

	```
	# curl -H "Host: my.kubernetes.example" 172.17.0.4/webapp1
	```

- The second request will be processed by the webapp2 deployment.

	```
	# curl -H "Host: my.kubernetes.example" 172.17.0.4/webapp2
	```

- Finally, all other requests will be processed by webapp3 deployment.

	```
	# curl -H "Host: my.kubernetes.example" 172.17.0.4
	```
