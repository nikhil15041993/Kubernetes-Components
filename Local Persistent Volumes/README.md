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

# How to Delete PV(Persistent Volume) and PVC(Persistent Volume Claim) stuck in terminating state?

 When you are planning to delete the Persistent Volume as well as Persistent Volume Claim then you must follow an order -

* First delete - Persistent Volume Claim
* Second delete- Persistent Volume


Delete Reclaim Policy- There are two types of volume provisioning in Kubernetes Static and Dynamic.

* Static Volume Provisioning - You have to manually configure the PV, PVC but when you need to delete both PV, PVC then you must start from PVC and then go for PV.
* Dynamic Volume Provisioning - You have to set up Storage Class along with PVC and Storage Class will take of dynamic provisioning of PV(Persistent Volume). If you delete the PVC(Persistent Volume Claim) then all the PV(Persistent Volume) provisioned dynamically will be deleted.


Each Kubernetes resource running in the cluster has Finalizers associated with it.

Finalizers prevent accidental deletion of resources(Persistent Volume, Persistent Volume Claim, POD, etc..)
If you accidentally issue kubectl delete command on Kubernetes resource and if there is a finalizer associated with that resource then it is going to put the resource in Read-Only mode and prevent it from deletion.

So if your Kubernetes resource such as Persistent Volume, Persistent Volume Claim, or POD is stuck in the terminating state then you must remove the Finalizer from that resource and after removing the Finalizer you should be able to delete the resource.


##  How to remove the Finalizer from Persistent Volume, Persistent Volume Claim or POD

To remove the Finalizer you have to use the ```kubectl patch command``` with the exact resource name.

```
kubectl patch pv <pv name> -p '{"metadata":{"finalizers":null}}'
```

Similarly, if your Persistent Volume Claim is stuck in Terminating state then run the following kubectl patch command to remove the Finalizer -
```
kubectl patch pvc <pvc name> -p '{"metadata":{"finalizers":null}}'
```

Now after patching both Persistent Volume Claim and Persistent Volume, you can delete both the resources.


This Same command  we can use on  storage class also
