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
