# Kubernetes - Running Stateful Services on Kubernetes

# Deploy NFS Server

- NFS is a protocol that allows nodes to read/write data over a network.
- The protocol works by having a master node running the NFS daemon and stores the data.
- This master node makes certain directories available over the network.
- Clients access the masters shared via drive mounts.
- From the viewpoint of applications, they are writing to the local disk.
- Under the covers, the NFS protocol writes it to the master.
- In this scenario, and for demonstration and learning purposes, the role of the NFS Server is handled by a customised container.
- The container makes directories available via NFS and stores the data inside the container.
- In production, it is recommended to configure a dedicated NFS Server.
- Start the NFS using the command 

	```
	# docker run -d --net=host --privileged --name nfs-server katacoda/contained-nfs-server:centos7 /exports/data-0001 /exports/data-0002
	```

- The NFS server exposes two directories, data-0001 and data-0002. In the next steps, this is used to store data.

# Deploy Persistent Volume

- For Kubernetes to understand the available NFS shares, it requires a PersistentVolume configuration.
- The PersistentVolume supports different protocols for storing data, such as AWS EBS volumes, GCE storage, OpenStack Cinder, Glusterfs and NFS.
- The configuration provides an abstraction between storage and API allowing for a consistent experience.
- In the case of NFS, one PersistentVolume relates to one NFS directory.
- When a container has finished with the volume, the data can either be Retained for future use or the volume can be Recycled meaning all the data is deleted.
- The policy is defined by the persistentVolumeReclaimPolicy option.
- For structure is:

```
apiVersion: v1
kind: PersistentVolume
metadata:
  name: <friendly-name>
spec:
  capacity:
    storage: 1Gi
  accessModes:
    - ReadWriteOnce
    - ReadWriteMany
  persistentVolumeReclaimPolicy: Recycle
  nfs:
    server: <server-name>
    path: <shared-path>
```

- The spec defines additional metadata about the persistent volume, including how much space is available and if it has read/write access.
- Create two new PersistentVolume definitions to point at the two available NFS shares.

	```
	# kubectl create -f nfs-0001.yaml
	# kubectl create -f nfs-0002.yaml
	```

- Click on the below links to get the definitions
	- [nfs-0001.yaml](https://github.com/LinuxLibrary/kubernetes/blob/master/Builds/KC-Builds/08-Stateful_Services/nfs-0001.yaml)
	- [nfs-0002.yaml](https://github.com/LinuxLibrary/kubernetes/blob/master/Builds/KC-Builds/08-Stateful_Services/nfs-0002.yaml)
