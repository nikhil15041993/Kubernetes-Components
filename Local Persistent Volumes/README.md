## Step 1: Create StorageClass with WaitForFirstConsumer Binding Mode
According to the docs, persistent local volumes require to have a binding mode of WaitForFirstConsumer. the only way to assign the volumeBindingMode to a persistent volume seems to be to create a storageClass with the respective volumeBindingMode and to assign the storageClass to the persistent volume. Let us start with

storageClass.yaml
```
kind: StorageClass
apiVersion: storage.k8s.io/v1
metadata:
  name: my-local-storage
provisioner: kubernetes.io/no-provisioner
volumeBindingMode: WaitForFirstConsumer
```
Create this volume by using following command
```
kubectl create -f storageClass.yaml
```
The output should be:
```
storageclass.storage.k8s.io/my-local-storage created
```


## Step 2: Create Local Persistent Volume
Since the storage class is available now, we can create local persistent volume with a reference to the storage class we have just created:

persistentVolume.yaml
```
apiVersion: v1
kind: PersistentVolume
metadata:
  name: my-local-pv
spec:
  capacity:
    storage: 500Gi
  accessModes:
  - ReadWriteOnce
  persistentVolumeReclaimPolicy: Retain
  storageClassName: my-local-storage
  local:
    path: /mnt/disk/vol1
  nodeAffinity:
    required:
      nodeSelectorTerms:
      - matchExpressions:
        - key: kubernetes.io/hostname
          operator: In
          values:
          - node1
```

The output should look like follows:
```
persistentvolume/my-local-pv created
```
## Step 3: Create a Persistent Volume Claim

Similar to hostPath volumes, we now create a persistent volume claim that describes the volume requirements. One of the requirement is that the persistent volume has the volumeBindingMode: WaitForFirstConsumer. We can assure this by referencing the previously created a storageClass:

persistentVolumeClaim.yaml

```
kind: PersistentVolumeClaim
apiVersion: v1
metadata:
  name: my-claim
spec:
  accessModes:
  - ReadWriteOnce
  storageClassName: my-local-storage
  resources:
    requests:
      storage: 500Gi
```
```
kubectl create -f persistentVolumeClaim.yaml
```

