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
- When a cluster user needs persistent storage in one of their pods, they first create a `PersistentVolumeClaim` object in which they either refer to a specific persistent volume by name, or specify the minimum volume size and access mode required by the application, and K8s find a `PersistentVolume` that meets these requirements. In both cases, the persistent volume is then bound to the claim and is given exclusive access. The claim can then be referenced in a volume definition within one or more pods. when the pod runs, the storage volume configured in the PersistentVolume object is attached to the worker node and mounted into the pod's containers.
## 8.2 Creating persistent volumes and claims
Let's revisit the quiz pod from the previous chapter. You may remember that this pod contains a `eksElasticBlockStore` volume. You'll modify that pod's manifest to make it use the `eksElasticBlockStore` via `PersistentVolume` object.  

There are usually two different types of Kubernetes users involved in the provisioning and use of persistent volumes. you will take both in the next exercises:
- Cluster administrator: create some persistent volumes, one of them will point to the existing `eksElasticBLockStore`. 
- Developer, user: create a `PersistentVolumeClaim` to get ownership of that volume and use it in the quiz pod.
## 8.2.1 Creating a PersistentVolume object
- If you use Google Kubernetes Engine to run these examples, you'll create persistent volumes that point to GCE Persistent Disks (GCE PD).
- If use other cloud provider, consult the provider's documentation to learn how to create the physical volume in their environment.
- If use Minikube, Kind, or other type of cluster, you don't need to create volumes because you'll use a persistent volumes that refers to a local directory on the worker node.
### Creating a persistent volume with GCE Persistent disk as the underlying storage
- In Google cloud, you can create GCE Persistent Disk name `quiz-data` by using this command:
```bash
$ gcloud compute disks create quiz-data
```
- After the disk is created, you then create a manifest file for the `PersistentVolume` object, here is the syntax:  

#### ** `pv.quiz-data.gcepd.yaml` **
```bash
apiVersion: v1
kind: PersistentVolume 
metadata:
  name: quiz-data # The name of this persistent volume
spec:
  capacity: # The storage capacity of this volume
    storage: 1Gi
  accessModes: # Whether a single node or many nodes can access this volume in read/write or read-only mode.
  - ReadWriteOnce
  - ReadOnlyMany
  gcePersistentDisk: # This persistent volume uses the GCE Persistent Disk created in the previous chapter
    pdName: quiz-data
    fsType: ext4

```
### Creating persistent volumes backed by other storage technologies
- If your Kubernetes cluster runs on a different cloud provider, You should be able to easily change the persistent volume manifest to use something other than a GCE Persistent Disk.
- If you used Minikube or Kind to provision your cluster, you can create a persistent volume that uses a local Directory on the worker node instead of network storage using `hostPath` field in the `PersistentVolume` manifest. The manifest for the `quiz-data` look like this:  
```bash
apiVersion: v1
kind: PersistentVolume
metadata:
  name: quiz-data
spec:
  capacity:
    storage: 1Gi
  accessModes:
  - ReadWriteOnce
  - ReadOnlyMany
  hostPath: # Instead of a GCE Persistent Disk, this persistent volume refers to a local directory on the host node 
    path: /var/quiz-data
```
### Specifying the volume capacity
- The `capacity` of the volume indicates the size of the underlying volume. Each persistent volume must specify its capacity so that Kubernetes can determine whether a particular persistent volume can meet the requirements specified in the persistent volume claim before it can bind them.
### Specifying volume access modes
- Each persistent volume must specify a list of `accessModes` it supports. Depending on the underlying technology, a persistent volume may or may not be mounted by multiple worker nodes simultaneously in read/write or read-only mode.
- Kubernetes inspects the persistent volume's access modes to determine if it meets the requirements of the claim
- test