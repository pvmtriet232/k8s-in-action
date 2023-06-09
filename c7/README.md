# Kubernetes in Action, 2nd Edition

## Chapter 7. Mounting storage volumes into the Pod's containers


### Fortune pod with or without a volume
- [fortune-no-volume.yaml](fortune-no-volume.yaml) - YAML manifest file for the `fortune-no-volume` pod
- [fortune-emptydir.yaml](fortune-emptydir.yaml) - YAML manifest file for the `fortune-emptydir` pod with an emptyDir volume
- [fortune.yaml](fortune.yaml) - YAML manifest file for the `fortune` pod with two containers that share a volume

### MongoDB pod with external volume
- [mongodb-pod-gcepd.yaml](mongodb-pod-gcepd.yaml) - YAML manifest file for the `mongodb` pod using a GCE Persistent Disk volume
- [mongodb-pod-aws.yaml](mongodb-pod-aws.yaml) - YAML manifest file for the `mongodb` pod using an AWS Elastic Block Store volume
- [mongodb-pod-nfs.yaml](mongodb-pod-nfs.yaml) - YAML manifest file for the `mongodb` pod using an NFS volume
- [mongodb-pod-hostpath.yaml](mongodb-pod-hostpath.yaml) - YAML manifest file for the `mongodb` pod using a hostPath volume (for use in _Minikube_)
- [mongodb-pod-hostpath-kind.yaml](mongodb-pod-hostpath-kind.yaml) - YAML manifest file for the `mongodb` pod using a hostPath volume (for use in _kind_)

### Using a hostPath volume
- [node-explorer.yaml](node-explorer.yaml) - YAML manifest file for the `node-explorer` pod
- [node-explorer-directory.yaml](node-explorer-directory.yaml) - YAML manifest file for the `node-explorer` pod with the hostPath volume type set to Directory


# K8s-in-action: Chap 07

## running the quiz service in a pod without a volume
```bash
$ k apply -f pod.quiz.novolume.yaml

$ k port-forward quiz 8080
```
## adding questions to the database
```bash 
$ k exec -it quiz -c mongo -- mongo

use kiada

db.questions.insert({
... id: 1,
... text: "what does k8s mean?",
... answers: ["Kates", "Kubernetes", "Kooba Dooba Doo!"],
... correctAnswerIndex: 1})

// to confirm questions that inserted are now stored in the database:

db.questions.find()

// to retrieve a random question through the Quiz API:

$ curl localhost:8080/questions/random
```
## Restarting the MongoDb database

MongoDb db >> container's filesystem, they are lost every time the container is restarted.
To confirm:
```bash
$ k exec -it quiz -c mongo -- mongo admin --eval "db.shutdownServer()"
```
The container is new, with a fresh filesystem, it doesn't contain the questions you entered. To confirm:
```bash
k exec -it quiz -c mongo -- mongo --quiet --eval "db.questions.count()"
> 0
```
what happened is the pod is still the same, the quiz-api container has been running fine this whole time. Only the mongo container was restarted (recreated)
### this is why you need volume to persist data in case of restarting container
## 7.1.2 Understanding how volumes fit into pods
Volumes are component within the pod and thus share its lifecycle.  

A volume is defined at the pod level and then mounted at the desired location in the container. 

The lifecycle of volume is tied to the lifecycle of the entire pod and is independent of the lifecycle of the container in which it is mounted. Due to this fact, volumes are also used to persist data across container restarts.
### Persisting files across container restarts
Volumes in a pod are created when the pod is set up - before containers are started. Volumes are torn down when the pod is shut down.  

Each time a container restarted, the volumes that the container is configured to use are mounted in the container's filesystem. App running in the container can read from the volume and write to it if the volume and mount are configured to be writable.  

### Mounting multiple volumes in a container
Why you want to mount multiple volumes in one container?  

Because these volumes may serve different purposes and can be of different types with different performance characteristics.  

In pods with more than one container , some volumes can be mounted in some containers but not in others . This is especially useful when a volume contains sensitive information that should only be accessible to some containers.
### Sharing file between multiple containers
A volume can be mounted in more than one container so that applications running in these containes can share files.  

For example, you create a pod with a web server and a content-producing agent both running in two different containers.  

Content-producing agent >> volume >> web-server >> user.  

The same volume can be mounted at different places in each container, depending on the needs of container itself.  

/var/data [read/write] mount to volume [read-only] mount to /var/html.  

Volume mounted in each container can be configured either as read/write or ad read-only.  

Best practise of security, prevent the web-server from writing to the volume or this will allow attacker to compromise the system if the web-server software has a vulnerability that allows attackers to write arbitraty files to the filesystem and execute them.  

Other examples of using a single volume in two containers are cases where a sidecar container runs a tool that processes or rotates the web server log or when an init container creates configurations files for the main app container.  
### Persisting data across pod instances

A volume die when the pod die, but depending on the volume type, the files in the volume can remain intact after the pod and volume die and can be later mounted into a new volume.  

The pod volume can map to persistent storage outside the pod, typically network-attached storage volume is thus persistent and can be used by the application event after the pod it runs in is replaced with a new pod running on a different worker node.  

### Sharing data between pods 
If you use a simple local directory on the worker node's filesystem, the pods in it can have map to that directory and share files.  

If the persistent storage is a network-attached storage volume, the pods may be able to use it even when they are deployed to different nodes. Depending on whether the underlying storage technology supports concurrently attaching the network volyme to multiple computers.  

Like NFS (Network File System) allow you to attach volume in read/write mode on multiple computers, other technologies available in cloud env, such as Google Compute Engine Persistent Disk...   
### Introducing the available volume types
When you add a volume to a pod, you must specify the volume type. A wide range of volume types is available. Here are some of the supported volume types:
- emptyDir : A simple directory that allows the pod to store data for the duration of its life cycle. The directory is created just before the pod starts and is initially empty - hence the name. 
- gitRepo : which is now deprecated, is initialized by cloning a Git repository. It's is recommended to use and emptyDir volume and init it using and init conatiner.
- hostPath : Used for mounting files from the worker node's filesystem into pod. 
- nfs : and NFS share mounted into the pod.
- gcePersistentDisk (Google Compute Engine Persistent Disk),
- awsElasticBlockStore (Amazon Web Services Elastic Block Store),
- azureFile, azureDisk
- cephfs, cinder, fc, flexVolume, flocker, glusterfs, iscsi, portworxVolume, quobyte, rbd, scaleIO, storageos, photonPersistentDisk, vsphereVolume : Used for mounting other types of network storage.
- configMap, secret, downwardAPI, projected : Special types of volumes used to expose information about the pod and other Kubernetes object through files. Used to configure the application running in the pod. Learn more in Chap 9
- persistentVolumeClaim : A portable way to integrate external storage into pods. Instead of pointing directly to an external storage volume, this volume type points to a PersistentVolumeClaim object that points to a PersistentVolume object that finally references the actual storage. Learn more in next chapter.
- csi : a pluggable way of adding storage via the Container Storage Interface. This volume type allows anyone to implement their own storage driver that is the referenced in the csi volume definition. DUsing pod setup, the CSI driver is called to attach the volume to the pod.   
These volume types serve different purposes. Learn more on next section.
## 7.2 Using an emptyDir volume
The simplest volume type is emptyDir. In a pod with two or more containers, an emptyDir volume is used to share data between them.
## 7.2.1 Persisting files across container restarts
### Adding an emptyDir volume to a pod
You edit the definition of the quiz pod so that the MongoDb process writes its files to the volume instead of the filesystem of the container it runs in.  

The steps is :  
- Add emptyDir volume to the pod
- Mount the volume to the container.  
### Configure the emptyDir volume
The emptyDir volume likes other volumes. It come with a few configuration options which is the sub-fields that allow you to configure the volume.  

Configuration options for an emptyDir volume:  
- medium: if left empty, default medium of the host node is used, other option is Memory, virtual memory filesystem where the files are kept in memory instead of on the hard disk.
- sizeLimit :the total amount of local storage required for the directory, whether on disk or in memory. (e.g : 10Mi)
### Mounting the volume to a container
### Table 7.2 Configure options for a volume mount

| Field | Description     |
| :-------- | :---------- |
| `name` | name of the volume to mount. Must match one of the volumes defined in the pod | 
| `mountPath` | the path within the container at which to mount the volume. |
| `readOnly` | Whether to mount the volume ad read-only. Defaults to false. |
| `mountPropagation` | Specfifies what should happen if additional filesystem volumes are mounted inside the volume. Defaults to None, which mearns that the container wont reveive any mounts that are mounted by the host, and the host wont receive any mounts that are mounted by the container. `HostTocontainer` means that the container will receive all mounts that are mounted into this volume by the host , but not the other way around. `Bidirectional` means that the container will receive mounts added by the host, and the host will receive mounts by the container |
| `subPath` | Default to "" which indicates that the entire volumes is to be mounted into the container. When set to a non-empty string , only the specified `subPath` within the volume is mounted into the container.|
| `subPathExpr` | just like `subPath` but can have environment variable references using the syntax $(ENV_VAR_NAME). Only environment variables that are explicitly defined in the container definition are applicable. Implicit variables such as HOSTNAME will not be resolved. You'll learn how to specify environment variables in chapter 9.|

In most case, you only need `name`, `mountPath`, whether the mount should be `readOnly`.  
`mountPropagation` option comes to play for advance use-cases where additional mounts are added to the volume's file tree later, either from the host or from the container. The `subPath` and `subPathExpr` options when mounting single volume with multi directories mount to different containers instead of using multiple volumes.  

The `subPathExpr` is used when a volume is shared by multiple pod replicas.
### Understanding the lifespan of an emptyDir volume
- Run pod.quiz.emptydir.yaml
- Insert question using the shell script `insert-question.sh`
- Check if the question was inserted to the mongo:
```bash
$ k exec -it quiz -c mongo -- mongo kiada --quiet --eval "db.questions.count()"
```
- Now, shutdown the server using:
```bash
$ k exec -it quiz -c mongo -- mongo admin --eval "db.shutdownserver()"
```
- Check the mongo container was restarted
- After restarting, check that the question still remain in the database:
```bash
$ k exec -it quiz -c mongo -- mongo kiada --quiet --eval "db.questions.count()"
```
The result should be "1" so that the data has survived the container restart.  
The question is where exactly the files are stored in volume?
### Understanding where the file in an emptyDir are stored
The files in an `emptyDir` volume are stored in a directory of the host node's filesystem.   
This directory is mounted into the container at the desired location.  

![Getting Started](./picture/pic1.png)

the `pod_UID` is the unique ID of the pod, To see the directory, run this command to get `pod_UID`:
```bash
k get pod quiz -o json | jq .metadate.uid
// "4f49f452-2a9a-4f70-8df3-31a227d020a1"
```
To get the name of the node that runs the pod use:  
```bash
$ k get po quiz -o wide
$ k get pod quiz -o json | jq .spec.nodeName
``` 
### Creating the emptyDir volume in memory
```bash
volumes:
    - name: app-volume
      emptyDir:
        medium: Memory #A
```
#A This directory will be stored in memory
Why you need `emptyDir` volume in memory?  
To store sensitive data, because the data is not written to disk, there is less chance that the data will be compromised and persisted longer than desired. 
### Specifying the size limit for the emptydir volume
Use `sizeLimit` field to configure the size of an `emptyDir` volume.  
## 7.2.2 Populating an emptyDir volume with data using an init container
The 1st way to do it: run MongoDb container locally, insert data , commit the container state into a new image and use that image in your pod.  
The better way: package data into a container image and copy the data files from the container to the volume when the container starts by using `init container`
- Build an image contain the insert-question.js script by busybox with command cp the file to the `/initdb.d` 
- declare an init container with that image and with an `emptyDir` volume mount at `/initdb.d` to inject the question to the volume (the script is now located in `initdb` volume, and init container is completed)
- mount the `initdb` volume to the mongo db at `/docker-entrypoint-initdb.d` directory so the container pick the script up and run it when starting. This inserts the questions into database in`/data/db` directory - mounted to `quiz-data` volume, so the data can now survive container restart.
```bash
apiVersion: v1
kind: pod
metadata:
    name: quiz
spec:
    volumes:
    - name: initdb
      emptyDir: {}
    - name: quiz-data
      emptyDir: {}
    initContainer:
    - name: installer
      image: luksa/quiz-initdb-script-installer:0.1
      volumeMounts:
      - name: initdb
        mountPath: /initdb.d
    containers:
    - name: quiz-api
      image: luksa/quiz-api:0.1
      ports:
      - name: http
        containerPort: 8080
    - name: mongo
      image: mongo
      volumeMounts:
      - name: quiz-data
        mountPath: /data/db
      - name: initdb
        mountPath: /docker-entrypoint-initdb.d/
        readOnly: true
```
## 7.2.3 Sharing files between containers
To share file between container, a volume need to be mounted into both containers.  
The processes will like this: [container quote-writer: shell-script] >> a new quote to the volume every 60s [READ-BY] [container nginx] serve via `http`
### Create a pod with two container and a shared volume
```bash
apiVersion: v1
kind: pod
metadata:
    name: quote
spec:
    volumes:
    - name: shared
      emptyDir: {}
    containers:
    - name: nginx
      image: alpine-nginx
      volumeMounts:
      - name: shared
        mountPath: /usr/share/nginx/html
        readOnly: true
    - name: quote-writer
      image: luksa/quote-writer:0.1
      volumeMounts:
      - name: shared
        mountPath: /var/local/output
      ports:
      - name: http
        containerPort:80
```
Container `quote-writer` write `quote` file to `/var/local/output`.  
Container `nginx` serves files from `/usr/share/nginx/html` directory.  
To verify that the quote has been written every 60s:
```bash
$ k port-forward quote 1080:80
$ curl localhost:1080/quote
```
Or
```bash
$ k exec quote -c quote-writer --cat /var/local/output/quote
// or
$ k exec quote -c nginx -- cat /usr/share/nginx/html/quote
```
## 7.3 Using external storage in pods
Why you need external storage?  
An `emptyDir` volume is a dedicated directory created for and used exclusively by the pod in which the volume is defined.  
When the pod is deleted, the volume and it's contents are deleted.  

The other types of volumes don't create a new directory, but instead mount an existing external directory in the filesystem of container.  
This kind of volume can be shared by multiple pods and can survive the instantiations of the pod.  

### Use an AWS Elastic BLock Store volume
If you run your cluster on AWS EC2, use can use `awsElasticBlockStore` volume, If You use azure, you can use `azurefile` or `azuredisk`
```bash
spec:
    volumes:
    - name: quiz-data
      awsElasticBlockStore: # The volume refers to an awsElasticBlockStore
        volumeID: quiz-data #The Id of the EBS volume
        fsType: ext4 #file system type
    containers:
    - ...
```
### Use an NFS volume
If your cluster runs on your own server, you can use NFS share by specifying the NFS server address and the exported path 
```bash
spec:
    volumes:
    - name: quiz-data
      nfs: # This volume refers to an NFS share.
        server: 1.2.3.4 # IP address of the NFS server
        path: /some/path # File path exported by the server
```
### Using other storage technologies
Other supported options are `iscsi`,`glusterfs`, `rbd` ,`flexVolume`, `cinder` ...   
To learn more:
```bash
$ k explain pod.spec.volumes
```
for detail:
```bash
$ k explain pod.spec.volumes.iscsi
```
### Why does k8s force software developers to understand low-level storage?
Because you can't use the same manifest without modification to deploy the pod in another cluster.  
## 7.3.3 Understanding how external volumes are mounted
...
## 7.4 Accessing files on the worker node's filesystem
Most pods shouldn't care which host node they are running on, and they shouldn't access any files on the node's filesystem.  
System-level pods are the exeption. They may need to read the node's files or use the node's filesystem to access the node's devices ...   

K8s makes this possible through the `hostPath` volume type.
### Introducing the hostPath volume
A `hostPath` volume points to a specific file or directory in the filesystems of the host node, ad shown in the next figure.  
Pods running on the same node and using the same path in their `hostPath` volume have access to the same files, whereas pods on other nodes do not.   
The most dangerous volume types in k8s and is usually reserved for use in privileged pods only.
## 7.4.2 Using a HostPath volume to explore the entire filesystem of the host node within the pod
```bash
volumes:
- name: host-root
  hostPath:
    path: /
    ...
```
Apply the pod.node-explorer.yaml  

```bash
$ k exec -it node-explorer -- sh

$ cd /host
```
Now you inside the root directory of the node's filesystem of a random node in your cluster 
if you'd like to deploy the pod on the specific node, edit `node-explorer.specific-node.pod.yaml` , set `.spec.nodeName` field to the name of the node you choose.
### Specifying the type for `hostPath` volume
| Type   |      Description      | 
|:----------|:-------------:|
| `Directory` | K8s checks if a dir exists at the specified path. You use this type if you want to mount a pre-existing dir into the pod and want to prevent the pod from running if the directory doesn't exist |
| `DirectoryOrCreate` |   Same as `Directory`, but if nothing exists at the specified path, an empty directory is created    |  
| `File` | The specified path must be a file. | 
| `FileOrCreate` | ... |
| `BlockDevice` | The Specified path must be a block device. |
| `CharDevice` | ... |
| `Socket` | The Specified path must be a UNIX socket. | . 

If the specified path doesn't match the type, the pod's containers don't run.


