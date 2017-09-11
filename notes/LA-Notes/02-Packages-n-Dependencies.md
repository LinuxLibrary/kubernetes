# Packages

- Install NTP on all the machines (master and nodes)

	```
	sudo yum install ntp -y
	```

- Enable and start `ntpd` service

	```
	sudo systemctl enable ntpd && sudo systemctl start ntpd
	```

- Edit the `/etc/hosts` file and add the host details of the all the machines
	
	```
	vi /etc/hosts
	```
	
	```
	192.168.1.50	master01
	192.168.1.51	node01
	192.168.1.52	node02
	192.168.1.53	node03
	```
	
- Create docker and kubernetes repo files

	```
	vi /etc/yum.repos.d/vag-docker.repo
	```

	```
	[docker]
	name=vag-docker
	baseurl=http://cbs.centos.org/repos/virt7-docker-common-release/x86_64/os/
	gpgcheck=0
	enabled=1
	```

- Check and disable the firewalls
- Install `kubernetes` and `docker`

	```
	yum install -y kubernetes docker
	```