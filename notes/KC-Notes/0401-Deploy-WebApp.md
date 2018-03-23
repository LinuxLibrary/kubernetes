# Kubernetes - Deploy Guestbook example on Kubernetes

- Launch a simple, multi-tier web application using Kubernetes and Docker.
- The Guestbook example application stores notes from guests in Redis via JavaScript API calls.
- Redis contains a master (for storage), and a replicated set of redis 'slaves'.

# Start Kubernetes

- To start we need a running Kubernetes cluster.
- To prepare a cluster [MultiNodeCluster](https://github.com/LinuxLibrary/kubernetes/blob/master/notes/KC-Notes/0201-Multi-Node-Cluster.md)
- Start a single-node cluster using the helper script. The helper script will launch the API, Master, a Proxy and DNS discovery. The Web App uses DNS Discovery to find the Redis slave to store data.
- See the 04-Scripts directory for the scripts used
- Start the launch script

	```
	# ./launch.sh
	```

- Check everything is up using the following health Check:

	```
	# kubectl cluster-info
	# kubectl get nodes
	```

- If the node returns NotReady then it is still waiting. Wait a couple of seconds before retrying.

# Redis Master Controller

- The first stage of launching the application is to start the Redis Master.
- A Kubernetes service deployment has, at least, two parts.
- A replication controller and a service.
- The replication controller defines how many instances should be running, the Docker Image to use, and a name to identify the service.
- Additional options can be utilized for configuration and discovery.
- If Redis were to go down, the replication controller would restart it on an active node.

***Create Replication Controller***
- In this example, the YAML defines a redis server called redis-master using the official redis running port 6379.
- The kubectl create command takes a YAML definition and instructs the master to start the controller.

	```
	# kubectl create -f redis-master-controller.yaml
	```

- To list out the replication controllers

	```
	# kubectl get rc
	```

- All containers described as Pods. A pod is a collection of containers that makes up a particular application, for example Redis. You can view this using kubectl

	```
	# kubectl get pods
	```

# Redis Master Service

- The second part is a service.
- A Kubernetes service is a named load balancer that proxies traffic to one or more containers.
- The proxy works even if the containers are on different nodes.
- Services proxy communicate within the cluster and rarely expose ports to an outside interface.
- When you launch a service it looks like you cannot connect using curl or netcat unless you start it as part of Kubernetes.
- The recommended approach is to have a LoadBalancer service to handle external communications.

***Create Service***
- The YAML defines the name of the replication controller, redis-master, and the ports which should be proxied.

	```
	# kubectl create -f redis-master-service.yaml
	```

- To list and describe services

	```
	# kubectl get services (or) kubectl get svc
	# kubectl describe services redis-master
	```

# Replication Slave Pods

- In this example we'll be running Redis Slaves which will have replicated data from the master.
- More details of Redis replication can be found at [Redis_Replication](http://redis.io/topics/replication)
- As previously described, the controller defines how the service runs.
- In this example we need to determine how the service discovers the other pods.
- The YAML represents the _GET_HOSTSFROM property as DNS.
- You can change it to use Environment variables in the yaml but introduces creation-order dependencies as the service needs to be running for the environment variable to be defined.

***Start Redis Slave Controller***
- In this case, we'll be launching two instances of the pod using the image kubernetes/redis-slave:v2. It will link to redis-master via DNS.

	```
	# kubectl create -f redis-slave-controller.yaml
	```

- To list the replication controllers

	```
	# kubectl get rc
	```

# Redis Slave Service

- As before we need to make our slaves accessible to incoming requests.
- This is done by starting a service which knows how to communicate with redis-slave.
- Because we have two replicated pods the service will also provide load balancing between the two nodes.

***Start Redis Slave Service***

	```
	# kubectl create -f redis-slave-service.yaml
	# kubectl get services
	```

# Frontend Replicated Pods

- With the data services started we can now deploy the web application.
- The pattern of deploying a web application is the same as the pods we've deployed before.

***Launch Frontend***
- The YAML defines a service called frontend that uses the image kubernetes/example-guestbook-php-redis:v2.
- The replication controller will ensure that three pods will always exist.

	```
	# kubectl create -f frontend-controller.yaml
	```

***List controllers and pods***

	```
	# kubectl get rc
	# kubectl get pods
	```
***PHP Code***

- The PHP code uses HTTP and JSON to communicate with Redis.
- When setting a value requests go to redis-master while read data comes from the redis-slave nodes.

# Guestbook Frontend Service

- To make the frontend accessible we need to start a service to configure the proxy.

***Start Proxy***
- The YAML defines the service as a NodePort.
- NodePort allows you to set well-known ports that are shared across your entire cluster.
- This is like -p 80:80 in Docker.
- In this case, we define our web app is running on port 80 but we'll expose the service on 30080.

	```
	# kubectl create -f frontend-service.yaml
	# kubectl get services
	```

# Access Guestbook Frontend

- With all controllers and services defined Kubernetes will start launching them as Pods.
- A pod can have different states depending on what's happening.
- For example, if the Docker Image is still being downloaded then the Pod will have a pending state as it's not able to launch.
- Once ready the status will change to running.

***View Pods Status***
- You can view the status using the following command:

	```
	# kubectl gget pods
	```

***Find NodePort***
- If you didn't assign a well-known NodePort then Kubernetes will assign an available port randomly.
- You can find the assigned NodePort using kubectl.

***View UI***
- Once the Pod is in running state you will be able to view the UI via port 30080.
- Under the covers the PHP service is discovering the Redis instances via DNS.
- You now have a working multi-tier application deployed on Kubernetes.
