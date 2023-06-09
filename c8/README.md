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
