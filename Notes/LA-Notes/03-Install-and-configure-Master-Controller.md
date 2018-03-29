# Install and configure Master Controller

- Go to `/etc/kubernetes` directory and edit the config

	```
	vi config
	```
	
	```
	KUBE_MASTER="--master=http://master01:8080"
	KUBE_ETCD_SERVERS="--etcd-servers=http://master01:2379"
	```

- Setup ETCD

	```
	vi /etc/etcd/etcd.conf
	```
	- Edit the below in the member section

	```
	ETCD_LISTEN_CLIENT_URLS="http://0.0.0.0:2379"
	```

	- Edit the below in the cluster section

	```
	ETCD_ADVERTISE_CLIENT_URLS="http://0.0.0.0:2379"
	```

- Setup the APISERVER

	```
	vi /etc/kubernetes/apiserver
	```

	```
	KUBE_API_ADDRESS="--address=0.0.0.0"
	KUBE_API_PORT="--port=8080"
	KUBELET_PORT="--kubelet-port=10250"
	```

	- Comment the KUBE_ADMISSION_CONTROL policies
	- Enable and start the below services

	```
	systemctl enable etcd kube-apiserver kube-controller-manager kube-scheduler
	systemctl start etcd kube-apiserver kube-controller-manager kube-scheduler
	```