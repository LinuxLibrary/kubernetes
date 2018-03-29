# Install and configure Minions

- Go to `/etc/kubernetes` directory and edit the config

	```
	vi config
	```
	
	```
	KUBE_MASTER="--master=http://master01:8080"
	KUBE_ETCD_SERVERS="--etcd-servers=http://master01:2379"
	```

- Edit and configure the kubelet configs

	```
	vi /etc/kubernetes/kubelet
	```

	```
	KUBELET_ADDRESS="--address=0.0.0.0"
	KUBELET_PORT="--port=10250"
	KUBELET_HOSTNAME="--hostname-override=node01"
	KUBELET_API_SERVER="--api-servers=http://master01:8080"
	```

	- Comment KUBELET_POD_INFRA_CONTAINER

- Enable and start the below services

	```
	systemctl enable kube-proxy kubelet docker
	systemctl start kube-proxy kubelet docker
	systemctl status kube-proxy kubelet docker | grep "(running)"
	```