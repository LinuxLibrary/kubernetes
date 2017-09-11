# Tags, Labels and Selectors

- Let us copy the existing yaml file for the lables demo

	```
	cp -prv nginx.yaml nginx-lables.yaml
	```

- Lets edit the new file and add a lable

	```
	vi nginx-lables.yaml
	```
	
	```
	apiVersion: v1
	kind: Pod
	metadata: 
	  name: nginx
	  lables:
	    app: nginx
	spec:
	  containers:
	  - name: nginx
	    image: nginx:1.7.9
	    ports: 
	    - containerPort: 80
	```

- If we create pods with the above template we can get the pods based on the lables

	```
	kubectl get pods -l app=nginx
	```