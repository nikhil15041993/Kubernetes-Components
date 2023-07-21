## RUNNING POSTGRES IN KUBERNETES WITH A PERSISTENT NFS VOLUME

Persistent NFS Volume for a Postgres local development Kubernetes Cluster
Prerequisites:

* A working Kubernetes Cluster
* A Linux computer or server to run a NFS server


  ## NFS Server Installation
If you don’t have an existing NFS Server, it is easy to create a local NFS server for our Kubernetes Cluster.

Install the Ubuntu needed packages and create a local directory /mnt/vagrant-kubernetes:

```
sudo apt install nfs-kernel-server
sudo mkdir -p /mnt/vagrant-kubernetes
sudo mkdir -p /mnt/vagrant-kubernetes/data
sudo chown nobody:nogroup /mnt/vagrant-kubernetes
sudo chmod 777 /mnt/vagrant-kubernetes
```

Edit the /etc/exports file to add the exported local directory and limit the share to the CIDR used by the Kubernetes Cluster nodes for external access.

```
/mnt/vagrant-kubernetes 192.168.50.0/24(rw,sync,no_subtree_check,insecure,no_root_squash)
```

Start the NFS server
```
# sudo exportfs -a
# sudo systemctl restart nfs-kernel-server    
# sudo exportfs -v
/mnt/vagrant-kubernetes
    192.168.50.0/24(rw,wdelay,insecure,no_root_squash,no_subtree_check,sec=sys,rw,insecure,no_root_squash,no_all_squash)
```



## Kubernetes Deployment File
The Postgres database deployment is composed of the following Kubernetes Objects:

* ConfigMap: stores common configuration data for the Postgres database server
* PersistentVolume: creates an NFS client that connects to the server and makes the NFS share available for volume claims.
* PersistentVolumeClaim: defines the characteristics of the needed volume, that Kubernetes will try to solve using an available PersistentVolume with the requested configuration.
* Deployment: starts an instance of a Postgres docker container using the supplied ConfigMap to redefine the value of Postgres configuration environment variables and mounts the PersistentVolumeClaim inside the container to replace PostgreSQL Data directory.
* Service (NodePort):  publishes the Postgres server port outside the Kubernetes Cluster.
(Full deployment file is shown at the end of the tutorial)


### Postgres Kubernetes ConfigMap
```
POSTGRES_DB: db (the database to be created at startup if it doesn’t exist)
POSTGRES_USER: user (the admin user that will be created)
POSTGRES_PASSWORD: pass (the password for the admin user that will be created)
PGDATA: /var/lib/postgresql/data/k8s (the location for the data to be used upon initialization, we will be using the subdirectory k8s for this instance)
```

Definition of ConfigMap in file postgresql.yaml
```
apiVersion: v1
kind: ConfigMap
metadata:
  name: psql-itwl-dev-01-cm
data:
  POSTGRES_DB: db
  POSTGRES_USER: user
  POSTGRES_PASSWORD: pass
  PGDATA: /var/lib/postgresql/data/k8s
```

## Postgres Kubernetes PersistentVolume
Defines an NFS PersistentVolume located at:

```
Server: 192.168.50.1 (The IP assigned by VirtualBox to our host)
Path: /mnt/vagrant-kubernetes/data (a subdirectory “data” inside the shared folder)
```

The volume has the following configuration:
```
Labels:
app: psql
ver: itwl-dev-01-pv
Capacity: It is able to store up to 1 Gigabyte.
Access Mode: Supports many clients reading and writing at the same time.
Retain policy: After usage, the volume is not destroyed
```

### Definition of PersistentVolume in file postgresql.yaml

```
apiVersion: v1
kind: PersistentVolume
metadata:
  name: psql-itwl-dev-01-pv
  labels: #Labels 
    app: psql
    ver: itwl-dev-01-pv 
spec:
  capacity:
    storage: 1Gi
  accessModes:
    - ReadWriteMany
  persistentVolumeReclaimPolicy: Retain
  nfs:
    server: 192.168.50.1
    path: "/mnt/vagrant-kubernetes/data"
```

### Postgres Kubernetes PersistentVolumeClaim

```
apiVersion: v1
kind: PersistentVolumeClaim
metadata:
  name: psql-itwl-dev-01-pvc
spec:
  selector:
    matchLabels:  #Select a volume with this labels
      app: psql
      ver: itwl-dev-01-pv
  accessModes:
    - ReadWriteMany
  resources:
    requests:
      storage: 1Gi
```

## Postgres Kubernetes Deployment

The container uses Postgres latest image from the public Docker registry (https://hub.docker.com/) and sets:

* Export container port 5432
* Set environment variables for the container using the Kubernetes config map psql-itwl-dev-01-cm
* Replace the directory /var/lib/postgresql/data inside the Postgres container with a volume named pgdatavol
* Define the pgdatavol volume as an instance of the persistentVolumeClaim psql-itwl-dev-01-pvc defined before

## Definition of Deployment in file postgresql.yaml

```
apiVersion: apps/v1
kind: Deployment
metadata:
  name: psql-itwl-dev-01
  labels: 
    app: psql 
    ver: itwl-dev-01
spec:
  replicas: 1
  selector:
    matchLabels:  #Deploy in a POD that has labels app: color and color: blue
      app: psql
      ver: itwl-dev-01
  template: #For the creation of the pod      
    metadata:
      labels:
        app: psql
        ver: itwl-dev-01
      annotations:
        sidecar.istio.io/inject: "false"        
    spec:
      containers:
        - name: postgres
          image: postgres:latest
          imagePullPolicy: "IfNotPresent"
          ports:
            - containerPort: 5432 
          envFrom:
            - configMapRef:
                name: psql-itwl-dev-01-cm          
          volumeMounts:
            - mountPath: /var/lib/postgresql/data
              name: pgdatavol
      volumes:
        - name: pgdatavol
          persistentVolumeClaim:
            claimName: psql-itwl-dev-01-pvc
```

### Postgres Kubernetes NodePort Service

Since our local Kubernetes Cluster doesn’t have a Cloud provided Load Balancer, we are using the NodePort functionality to access published ports in containers.

The NodePort will publish Postgres in port 30100 of every Kubernetes Master and Nodes:

192.168.50.11:30100
192.168.50.12:30100
192.168.50.13:30100


### Definition of Service in file postgresql.yaml:

```
apiVersion: v1
kind: Service
metadata:
  name: postgres-service-np
spec:
  type: NodePort
  selector:
    app: psql
  ports:
    - name: psql
      port: 5432        # Cluster IP http://10.109.199.234:port (docker exposed port)
      nodePort: 30100   # (EXTERNAL-IP VirtualBox IPs) 192.168.50.11:nodePort 192.168.50.12:nodePort 192.168.50.13:nodePort
      protocol: TCP
```




### Full file postgresql.yaml with all the resources to be deployed in Kubernetes:


```
apiVersion: v1
kind: ConfigMap
metadata:
  name: psql-itwl-dev-01-cm
data:
  POSTGRES_DB: db
  POSTGRES_USER: user
  POSTGRES_PASSWORD: pass
  PGDATA: /var/lib/postgresql/data/k8s

---
apiVersion: v1
kind: PersistentVolume
metadata:
  name: psql-itwl-dev-01-pv
  labels: #Labels 
    app: psql
    ver: itwl-dev-01-pv 
spec:
  capacity:
    storage: 1Gi
  accessModes:
    - ReadWriteMany
  persistentVolumeReclaimPolicy: Retain
  nfs:
    server: 192.168.50.1
    path: "/mnt/vagrant-kubernetes/data"
          
---

apiVersion: v1
kind: PersistentVolumeClaim
metadata:
  name: psql-itwl-dev-01-pvc
spec:
  selector:
    matchLabels:  #Select a volume with this labels
      app: psql
      ver: itwl-dev-01-pv
  accessModes:
    - ReadWriteMany
  resources:
    requests:
      storage: 1Gi
---

apiVersion: apps/v1
kind: Deployment
metadata:
  name: psql-itwl-dev-01
  labels: 
    app: psql 
    ver: itwl-dev-01
spec:
  replicas: 1
  selector:
    matchLabels:  #Deploy in a POD that has labels app: color and color: blue
      app: psql
      ver: itwl-dev-01
  template: #For the creation of the pod      
    metadata:
      labels:
        app: psql
        ver: itwl-dev-01
      annotations:
        sidecar.istio.io/inject: "false"        
    spec:
      containers:
        - name: postgres
          image: postgres:latest
          imagePullPolicy: "IfNotPresent"
          ports:
            - containerPort: 5432 
          envFrom:
            - configMapRef:
                name: psql-itwl-dev-01-cm          
          volumeMounts:
            - mountPath: /var/lib/postgresql/data
              name: pgdatavol
      volumes:
        - name: pgdatavol
          persistentVolumeClaim:
            claimName: psql-itwl-dev-01-pvc

---

apiVersion: v1
kind: Service
metadata:
  name: postgres-service-np
spec:
  type: NodePort
  selector:
    app: psql
  ports:
    - name: psql
      port: 5432        # Cluster IP http://10.109.199.234:port (docker exposed port)
      nodePort: 30100   # (EXTERNAL-IP VirtualBox IPs) http://192.168.50.11:nodePort/ http://192.168.50.12:nodePort/ http://192.168.50.13:nodePort/
      protocol: TCP

```

### Create the resources in Kubernetes
```
$ kubectl apply -f postgresql.yaml 

configmap/psql-itwl-dev-01-cm created
persistentvolume/psql-itwl-dev-01-pv created
persistentvolumeclaim/psql-itwl-dev-01-pvc created
deployment.apps/psql-itwl-dev-01 created
service/postgres-service-np created
```

### Check Kubernetes
Check that all resources have been created:
```
kubectl get all
```

### Test the Kubernetes Postgres database

Use the Postgres client psql (
```
apt-get install -y postgresql-client
```
)

```
 psql db_name -h 192.168.50.11 -p 30100 -U user_name
```


### Create a Table and add data

```
CREATE TABLE COLOR(ID SERIAL PRIMARY KEY   NOT NULL,NAME           TEXT     NOT NULL UNIQUE,RED            SMALLINT NOT NULL,GREEN          SMALLINT NOT NULL,BLUE           SMALLINT NOT NULL);
```

```
INSERT INTO COLOR (NAME,RED,GREEN,BLUE) VALUES('GREEN',0,128,0);
INSERT INTO COLOR (NAME,RED,GREEN,BLUE) VALUES('RED',255,0,0);
```

```
SELECT * FROM COLOR;
```

### Delete and Recreate the Postgres Instance
You can now destroy the Postgres instance and create it again, the data is preserved.

```
$ kubectl delete -f postgresql.yaml 
configmap "psql-itwl-dev-01-cm" deleted
persistentvolume "psql-itwl-dev-01-pv" deleted
persistentvolumeclaim "psql-itwl-dev-01-pvc" deleted
deployment.apps "psql-itwl-dev-01" deleted
service "postgres-service-np" deleted

$ kubectl apply -f postgresql.yaml 
configmap/psql-itwl-dev-01-cm created
persistentvolume/psql-itwl-dev-01-pv created
persistentvolumeclaim/psql-itwl-dev-01-pvc created
deployment.apps/psql-itwl-dev-01 created
service/postgres-service-np created
```

Use the psql client to check that the database tables have been preserved:

```
psql db -h 192.168.50.11 -p 30100 -U user
```
