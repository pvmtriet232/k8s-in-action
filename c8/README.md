# Chapter 8: Persisting data in PersistentVolume
In this Chapter You'll learn:
- Using PersistentVolume objects to represent persisten storage
- Claiming persistent volumes with PersistenVolumeClaim objects
- Dynamic provisioning of persistent volumes
- Using node-local persistent storage
## 8.1 Decouping pods from the underlying storage technology

- A developer who deploys apps on k8s shouldn't need to know what storage technology the cluster provides, 
- When you deploy an app to k8s, you don't refer directly the external storage in the pod manifest. Instead, you use an idirect approach...
- The volume definition in the pod manifest contains the IP address of the NFS server and the file path exported by that server. This ties the pod definition to a specific cluster and prevents it from being used elsewhere.
## 8.1.1 Introduction persistent volumes and claims
- To make the pod manifests portable across different cluster environments, the env-specific information about the actual storage volume is moved to a PersistentVolume object. 

- The pod's volume definition rever to a `PersistentVolumeClaim` object, the `PersistentVolumeClaim` is bound to a `PersistentVolume` object, the `PersistentVolume` object specifies the location and type of the network storage.

### What is Persistent volumes
- Like the name suggests, A `PersistentVolume` object represents a storage volume that is used to persist application data. 

- The `PersistentVolume object stores the information about the underlying storages and decouples this information from the pod

### What is Persistent volume claims
- A `PersistenVolumeClaim` object represents a user's claim on the persistent volume. Because its lifecycle is not tied to that of the pod, it allows the ownership of persistent volume to be decoupled from the pod. Before a user can use a persistent volume in their pods, they must create a `PersistentVolumeClaim` object. They can delete the pod at any time, and they won't lose ownership of the persistent volume.
### Using a persistent volume claim in a pod
- To use the persistent volume in a pod, in its manifest you simply refer to the name of the persistent volume claim that the volume is bound to.
- e.g you create a persistent volume claim that gets bound to a persistent volume that represents an NFS file share, you can attach the NFS file share to your pod by adding a volume definition that points to the `PersistentVolumeClaim` object.
### Using a claim in multiple pods
- Multiple pods in different nodes can use the same storage volume if they refer to the same `PersistentVolumeClaim` and therefore transitively to the same persistent volume.
- Depending on the underlying technology supports attaching the storage to many nodes concurrently, it can be used by the pods on different nodes. If not, the pods must all be scheduled to the node that attached the storage volume first.
## 8.1.2 What are the benefits of using persistent volumes and claims
- The biggest advantage of using persistent volumes and claims is that the infrastructure-specific details are now decoupled from the application represented by the pod.  
Works like this:
- Cluster administrator sets up (`PersistentVolume` points to the `NFS File Share`) 
- User create (`PersistentVolumeClaim` that represents the application's request for storage), then user create a pod that refer to `PersistenVolumeClaim`, when the pod run the Network storage volume(`NFS File Share`) is mounted into the pod's containers
- Instead of the developer adding a technology-specific volume to their pod, the cluster administrator sets up the underlying storage and then registers it in Kubernetes by creating a `PersistentVolume` object through Kubernetes API
- When a cluster user needs persistent storage in one of their pods, they first create a `PersistentVolumeClaim` object in which they either refer to a specific persistent volume by name, or specify the minimum volume size and access mode required by the application, and K8s find a `PersistentVolume` that meets these requirements. 
