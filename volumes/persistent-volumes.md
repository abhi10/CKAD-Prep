#h1 High Level Overview:
- Persistent Volume Definiton
- Different Volume Types


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

> k create -f pv-definition.yaml
> k get persistentVolume
