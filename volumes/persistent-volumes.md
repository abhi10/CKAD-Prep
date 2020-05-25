## High Level Overview:
- Persistent Volume Definiton
- Different Volume Types

**Definition:**
A *PersistentVolume (PV)* is a piece of storage in the cluster that has been provisioned by an administrator or dynamically provisioned using Storage Classes. It is a resource in the cluster just like a node is a cluster resource. PVs are volume plugins like Volumes, but have a lifecycle independent of any individual Pod that uses the PV. This API object captures the details of the implementation of the storage, be that NFS, iSCSI, or a cloud-provider-specific storage system

pv-definition.yaml
```
apiVersion: v1
kind: PersistenVolume
metadata:
  name: pv-voll
spec:
  accessModes:
    - ReadWriteOnce
  capacity:
    storage: 10Gi
  awsElasticBlockStore:
    volumeID: <volume-id>
    fsType: ext4
```
Commands for Persistent Volume:

> k create -f pv-definition.yaml   
> k get persistentVolume   
> k get persistenvolumeClaim   
> k delete persistentvolumeClaim <claim-name>


### Persistent Volume Claim:
A *PersistentVolumeClaim (PVC)* is a request for storage by a user. It is similar to a Pod. Pods consume node resources and PVCs consume PV resources. Pods can request specific levels of resources (CPU and Memory). **Claims can request specific size and access modes (e.g., they can be mounted once read/write or many times read-only).**

**Pointers:**
- PV and PVC are two different objects in Kubernetes.
- Every PVC is bound to a single PV.
- Binding is accomplished by Kubernetes using Labels and Selectors.
- PV has a default Reclaim Policy unless manually deleted.
  - persistenVolumeReclaimPolicy: Retain
  - persistenVolumeReclaimPolicy: Recycle (data is scrubbed before available)

**PVC Pod Definition:**
```
apiVersion: v1
kind: PersistentVolumeClaim
metadata:
  name: myclaim
spec:
  accessModes:
    - ReadWriteOnce
  resources:
    requests:
      storage: 500Mi

```

**Using PVC in a Pod Definition**
```
apiVersion: v1
kind: Pod
metadata:
  name: mypod
spec:
  containers:
    - name: myfrontend
      image: nginx
      volumeMounts:
      - mountPath: "/var/www/html"
        name: mypd
  volumes:
    - name: mypd
      persistentVolumeClaim:
        claimName: myclaim
```

Labs
====
- kubectl exec webapp cat /log/app.log
- How does bounding happen beween pvc and pv.
- How to limit bounding between them.
-
