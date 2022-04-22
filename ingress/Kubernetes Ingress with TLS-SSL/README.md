# Kubernetes Ingress with TLS/SSL

configuration for Ingress and HTTPS/TLS/SSL in Kubernetes.

Many ingress controllers are supported for Kubernetes:

* Nginx Controller
* HAProxy Ingress, Contour, Citrix Ingress Controller
* API Gatways like Traeffic, Kong and Ambassador
* Service mesh like Istio
* Cloud managed ingress controllers like Application Gateway Ingress Controller (AGIC), AWS ALB Ingress Controller, Ingress GCE

## Table of Content

* Installing an ingress controller (NGINX) into Kubernetes.
* Deploying 2 different applications/services.
* Configuring ingress to route traffic depending on the URL.
* configuring SSL/TLS using Cert Manager

## Step 1 Installing an ingress controller (NGINX) into Kubernetes.

### Create a namespace for the apps

```
kubectl apply -f app-namespace.yaml
```

### Deploy the 2 sample apps into Kubernetes

```
kubectl apply -f app1-deploy-svc.yaml 
kubectl apply -f app2-deploy-svc.yaml
```

Get the 2 public IP addresses 
```
kubectl get services --namespace app
```

### Add the Helm chart for Nginx Ingress

```
helm repo add ingress-nginx https://kubernetes.github.io/ingress-nginx
helm repo update
```

### Install the Helm (v3) chart for nginx ingress controller

```
helm install app-ingress ingress-nginx/ingress-nginx \
     --namespace ingress \
     --create-namespace \
     --set controller.replicaCount=2 \
     --set controller.nodeSelector."beta\.kubernetes\.io/os"=linux \
     --set defaultBackend.nodeSelector."beta\.kubernetes\.io/os"=linux
 ```
 
 ### Get the Ingress Controller public IP address
 
```
 kubectl get services --namespace ingress
```
Update the service type to ClusterIP instead of LoadBalancer

### Deploy the Ingress resource into Kubernetes

```
kubectl apply -f app-ingress.yaml 
```


## Step 2 configuring SSL/TLS using Cert Manager

### Create a namespace for Cert Manager

```
Create a namespace for Cert Manager
```

### Get the Helm Chart for Cert Manager

```
helm repo add jetstack https://charts.jetstack.io
helm repo update
```

### Install Cert Manager using Helm charts

```
helm install cert-manager jetstack/cert-manager --namespace cert-manager --set installCRDs=true --create-namespace --version v1.0.2
```

###  Check the created Pods

```
kubectl get pods --namespace cert-manager

```

### Install the Cluster Issuer

```
kubectl apply --namespace app -f ssl-tls-cluster-issuer.yaml
```

### Install the Ingress resource configured with TLS/SSL

 ```
 kubectl apply --namespace app -f ssl-tls-ingress.yaml
```

### Verify that the certificate was issued

```
kubectl describe cert app-web-cert --namespace app
```
