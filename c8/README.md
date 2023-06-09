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
- To use the persistent volume in a pod