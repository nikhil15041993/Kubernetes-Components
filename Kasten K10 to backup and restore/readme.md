There are three steps involved to use Kasten K10 backup and restore

* Installing Kasten K10 on your  Kubernetes cluster
* Installing MongoDB
* Backup and restore workflow using Kasten K10


## Step 1: Installing Kasten K10 on Your Cluster

We will use Kasten K10 Helm chart to install K10 on a Kubernetes cluster and to add the repo, we will use the following command:
```
helm repo add kasten https://charts.kasten.io/
```

Next we will create a Kasten namespace to deploy the K10 application:
```
kubectl create namespace kasten-io
```
Now we will install K10 using the command below:
```
helm install k10 kasten/k10 -n kasten-io
```
Helm install will create multiple deployments and services, and you can watch the status and validate the install by the following command:
```
kubectl get pods -n kasten-io --watch
```
Once the pods are in running condition, you can port-forward the service to access K10 dashboard from the browser.

You can access the K10 dashboard at 127.0.0.1:8080/k10/#/ after running the following command:
```
kubectl --namespace kasten-io port-forward service/gateway 8080:8000
```


## Step 2: Installing MongoDB on Cluster

We start with creating MongoDB namespace by using the following command:
```
kubectl create namespace mongodb
namespace/mongodb created
```
Then we will add MongoDB repo by following this command:
```
helm repo add bitnami https://charts.bitnami.com/bitnami
```
Now we will install MongoDB in the MongoDB namespace:
```
helm install mongodb bitnami/mongodb -n mongodb
```
This will provision Persistent Volume, Persistent Volume Claim, ReplicaSet, Deployment, and pod.

To validate the installation:

```
kubectl get all -n mongodb
```

We also have a Persistent Volume claim of 8gb attached to MongoDB pod to persist data over container restarts. MongoDB will automatically use DO block storage as a Persistent Volume.

We can verify that by:
```
kubectl get pvc -n mongodb
```

K10 automatically discovers MongoDB instances, and you will see the data and associated artifacts for this instance.

## Step 3: Backup and Restore using Kasten K10

Following command will show us the storage class name that is attached to MongoDB instance:
```
kubectl get pvc -n mongodb
```
Now we can annotate the do-block-storage by the following command:

```
kubectl annotate volumesnapshotclass do-block-storage
k10.kasten.io/is-snapshot-class=true
```
### Create a restore point (Manual Backup)

On the MongoDB instance, click “snapshot” to manually backup.

On the dashboard, you can see the progress of the backup.

Wait for the operation to complete, then you will see one restore point available.

### Restore
Click on the restore point you want to restore to. If there is more than the one, you will see all the restore points here. Clicking on Today, restore point will start operation to restore.

You will see the process in the dashboard as it starts the restoring process.

### Using Backup Policies

We have seen the workflow of backup using the manual method, another method is to use the backup policies to make daily, weekly, or hourly backups. You can set the backup and snapshot retention schedule independently for detailed control over how often backups are performed and the amount of total storage they consume.

Click on create policies. You will see the policy creation sidebar. Policies are extremely configurable, and multiple options are available.

When a policy is applied to the application, this executes a backup for the first time. The application’s compliance with the policy is reported in the application card. You will see the application is then compliant with all the policies.
