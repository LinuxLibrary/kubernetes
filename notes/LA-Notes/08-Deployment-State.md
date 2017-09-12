# Deployment State

- Copy an existing template and edit it as following

	```
	apiVersion: extensions/v1beta1
	kind: Deployment
	metadata:
	  name: nginx-deployment-prod
	spec: 
	  replicas: 1
	  template:
	    metadata:
	      labels:
	        app: nginx-deployment-prod
	    spec:
	      containers:
	      - name: nginx
	        image: nginx:1.7.9
	        ports: 
	        - containerPort: 80
	```

- Now create a deployment

	```
	kubectl create -f nginx-deployment-prod.yml
	```

- To view the deployments

	```
	kubectl get deployments
	```

- To view the pods

	```
	kubectl get pods
	```

	- We should notice the name of the pod with an extension to the deployment name. Which is to uniqly identify the pod
- Now let us create a dev deployment. For this let us copy the prod deployemnt template and edit as per our requirement

	```
	cp -prv nginx-deployment-prod.yaml nginx-deployment-dev.yaml
	```

	```
	apiVersion: extensions/v1beta1
	kind: Deployment
	metadata:
	  name: nginx-deployment-dev
	spec:
	  replicas: 1
	  template:
	    metadata:
	      labels:
	        app: nginx-deployment-dev
	    spec:
	      containers:
	      - name: nginx
	        image: nginx:1.7.9
	        ports:
	        - containerPort: 80
	```

- Now we can launch this deployment as well.