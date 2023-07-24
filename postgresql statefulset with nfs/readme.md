

## pv.yaml

```
apiVersion: v1
kind: PersistentVolume
metadata:
  name: pv-nfs-pv0
  labels:
    type: local
spec:
  storageClassName: manual
  capacity:
    storage: 200Mi
  accessModes:
    - ReadWriteOnce
  nfs:
    server: 13.127.108.92
    path: "/mnt/vagrant-kubernetes/node1"
---
apiVersion: v1
kind: PersistentVolume
metadata:
  name: pv-nfs-pv1
  labels:
    type: local
spec:
  storageClassName: manual
  capacity:
    storage: 200Mi
  accessModes:
    - ReadWriteOnce
  nfs:
    server: 13.127.108.92
    path: "/mnt/vagrant-kubernetes/node2"

```



## postgres.yaml

```
apiVersion: v1
kind: Service
metadata:
  name: nginx-headless
  labels:
    run: nginx-sts-demo
spec:
  ports:
  - port: 5432
    name: web
  clusterIP: None
  selector:
    run: nginx-sts-demo
---
apiVersion: v1
kind: Service
metadata:
  name: backend
  labels:
    run: backend
spec:
  ports:
  - port: 5432
    name: web
  selector:
    run: nginx-sts-demo

---

apiVersion: apps/v1
kind: StatefulSet
metadata:
  name: nginx-sts
spec:
  serviceName: "nginx-headless"
  replicas: 2
  #podManagementPolicy: Parallel
  selector:
    matchLabels:
      run: nginx-sts-demo
  template:
    metadata:
      labels:
        run: nginx-sts-demo
    spec:
      containers:
      - name: database-techibens 
        image: postgres:latest
        env:
        - name: POSTGRES_USER
          value: test
        - name: POSTGRES_PASSWORD
          value: test
        - name: POSTGRES_DB
          value: test
        - name: PGDATA
          value: /data/pgdata
        volumeMounts:
        - name: www
          mountPath: /data

  volumeClaimTemplates:
  - metadata:
      name: www
    spec:
      storageClassName: manual
      accessModes:
        - ReadWriteOnce
      resources:
        requests:
          storage: 100Mi
      ```
