# Set Up an Nginx Ingress on Kubernetes Using Helm


## Prerequisites

* cluster with your connection configuration configured as the kubectl default
* The Helm 3 package manager installed on your local machine

## Step 1 — Setting Up Deployments.yml file

The first deployment configuration will be in a file named hello-kubernetes-first.yaml. Create it using a text editor:
```
vi hello-kubernetes-first.yaml
```
```
apiVersion: v1
kind: Service
metadata:
  name: hello-kubernetes-first
spec:
  type: ClusterIP
  ports:
  - port: 80
    targetPort: 8080
  selector:
    app: hello-kubernetes-first
---
apiVersion: apps/v1
kind: Deployment
metadata:
  name: hello-kubernetes-first
spec:
  replicas: 3
  selector:
    matchLabels:
      app: hello-kubernetes-first
  template:
    metadata:
      labels:
        app: hello-kubernetes-first
    spec:
      containers:
      - name: hello-kubernetes
        image: paulbouwer/hello-kubernetes:1.8
        ports:
        - containerPort: 8080
        env:
        - name: MESSAGE
          value: Hello from the first deployment!
          
 ```
 
 Save and close the file.

Then, create this first variant of the hello-kubernetes app in Kubernetes by running the following command:
```
kubectl create -f hello-kubernetes-first.yaml
```

To verify the Service’s creation, run the following command:
```
kubectl get service hello-kubernetes-first
```
The output will be the following:
```
Output
NAME                     TYPE        CLUSTER-IP      EXTERNAL-IP   PORT(S)   AGE
hello-kubernetes-first   ClusterIP   10.245.124.46   <none>        80/TCP    7s
```

Open a file called hello-kubernetes-second.yaml for editing:
```
vi hello-kubernetes-second.yaml
```

```
apiVersion: v1
kind: Service
metadata:
  name: hello-kubernetes-second
spec:
  type: ClusterIP
  ports:
  - port: 80
    targetPort: 8080
  selector:
    app: hello-kubernetes-second
---
apiVersion: apps/v1
kind: Deployment
metadata:
  name: hello-kubernetes-second
spec:
  replicas: 3
  selector:
    matchLabels:
      app: hello-kubernetes-second
  template:
    metadata:
      labels:
        app: hello-kubernetes-second
    spec:
      containers:
      - name: hello-kubernetes
        image: paulbouwer/hello-kubernetes:1.8
        ports:
        - containerPort: 8080
        env:
        - name: MESSAGE
          value: Hello from the second deployment!
```

Now create it in Kubernetes with the following command:
```
kubectl create -f hello-kubernetes-second.yaml
```

Verify that the second Service is up and running by listing all of your services:
```
kubectl get service
```
The output will be similar to this:
```
Output
NAME                      TYPE        CLUSTER-IP       EXTERNAL-IP   PORT(S)   AGE
hello-kubernetes-first    ClusterIP   10.245.124.46    <none>        80/TCP    49s
hello-kubernetes-second   ClusterIP   10.245.254.124   <none>        80/TCP    10s
kubernetes                ClusterIP   10.245.0.1       <none>        443/TCP   65m
```



## Step 2 — Installing the Kubernetes Nginx Ingress Controller

To install the Nginx Ingress Controller to your cluster, you’ll first need to add its repository to Helm by running:

```
helm repo add ingress-nginx https://kubernetes.github.io/ingress-nginx
```
The output will be:
```
Output
"ingress-nginx" has been added to your repositories
```

Update to let Helm know what it contains:
```
helm repo update
```
Finally, run the following command to install the Nginx ingress:
```
helm install nginx-ingress ingress-nginx/ingress-nginx --set controller.publishService.enabled=true
```
This command installs the Nginx Ingress Controller from the stable charts repository, names the Helm release nginx-ingress, and sets the publishService parameter to true.

The output will be the following:
```
Output
NAME: nginx-ingress
LAST DEPLOYED: Fri Apr  3 17:39:05 2020
NAMESPACE: default
STATUS: deployed
REVISION: 1
TEST SUITE: None
NOTES:
```
You can watch the Load Balancer become available by running:
```
kubectl --namespace default get services -o wide -w nginx-ingress-ingress-nginx-controller
```

## Step 3 — Exposing the App Using an Ingress

Now you’re going to create an Ingress Resource and use it to expose the hello-kubernetes app deployments at your desired domains. You’ll then test it by accessing it from your browser.

You’ll store the Ingress in a file named hello-kubernetes-ingress.yaml. Create it using your editor:
```
vi hello-kubernetes-ingress.yaml
```
```
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: hello-kubernetes-ingress
  annotations:
    kubernetes.io/ingress.class: nginx
spec:
  rules:
  - host: "hw1.your_domain_name"
    http:
      paths:
      - pathType: Prefix
        path: "/"
        backend:
          service:
            name: hello-kubernetes-first
            port:
              number: 80
  - host: "hw2.your_domain_name"
    http:
      paths:
      - pathType: Prefix
        path: "/"
        backend:
          service:
            name: hello-kubernetes-second
            port:
              number: 80
              
```
You define an Ingress Resource with the name hello-kubernetes-ingress. Then, you specify two host rules, so that hw1.your_domain is routed to the hello-kuNext, you’ll need to ensure that your two domains are pointed to the Load Balancer via A records. This is done through your DNS provider.bernetes-first Service, and hw2.your_domain is routed to the Service from the second deployment (hello-kubernetes-second)


Next, you’ll need to ensure that your two domains are pointed to the Load Balancer via A records. This is done through your DNS provider.


Create it in Kubernetes by running the following command:
```
kubectl apply -f hello-kubernetes-ingress.yaml
```
You can now navigate to ```hw1.your_domain``` in your browser.
