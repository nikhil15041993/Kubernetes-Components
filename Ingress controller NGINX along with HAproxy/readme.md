# Setting up Ingress controller NGINX along with HAproxy for Microservice deployed inside Kubernetes cluster

![Alt text](https://jhooq.com/wp-content/uploads/2020/07/Screenshot-2020-07-28-at-16.23.14-1024x634.jpg)

## 1. Setup kubernetes Cluster
The first and most essential step up for you is - "You should have a kuberentes cluster setup and it should be running. "

Once you setup your kubernetes cluster you can run the following kubectl command to verify your cluster

$ kubectl get all
```
You should see default kubernetes service running as ClusterIP

```
```
NAME                 TYPE        CLUSTER-IP   EXTERNAL-IP   PORT(S)   AGE
service/kubernetes   ClusterIP   10.233.0.1   <none>        443/TCP   31m
 ```
 
 
## 2. Install/setup HAProxy on kubernetes node
Before we deep dive into the Kubernetes Ingress controller, lets complete our first and most important per-requisite of installing HAProxy Loadbalancer.

First update your package information using following command

Ubuntu
```
$ sudo apt-get update
```
CentOS
```
$ sudo yum check-update
```
You can use the following command to install HAProxy Loadbalancer

Ubuntu -
```
$ sudo apt-get -y install haproxy
```
CentOS
```
$ sudo yum install haproxy
```
Verify the installation - After the successful installation you should be able to see haproxy.cfg at /etc/haproxy/haproxy.cfg

You can check the installation version also -
```
$ haproxy version 
 HA-Proxy version 1.8.8-1ubuntu0.11 2020/06/22 
```
 
## 3. Update frontend, backend configuration of haproxy.cfg (/etc/haproxy/haproxy.cfg)

After the installation now you need to update the frontend as well as backend configuration for HAProxy.

Frontend - It receives the requests from the clients.
```
frontend Local_Server
    bind *:80
    mode http
    default_backend k8s_server
```
Backend - It is responsible for fulfilling the request sent from Frontend. Look at the following Backend configuration -
```
backend k8s_server
    mode http
    balance roundrobin
    server web1.example.com  100.0.0.2:8080
```

If you are wondering where to find the haproxy.cfg then use the following command

```
$ sudo vi /etc/haproxy/haproxy.cfg


global
        log /dev/log    local0
        log /dev/log    local1 notice
        chroot /var/lib/haproxy
        stats socket /run/haproxy/admin.sock mode 660 level admin expose-fd listeners
        stats timeout 30s
        user haproxy
        group haproxy
        daemon

        # Default SSL material locations
        ca-base /etc/ssl/certs
        crt-base /etc/ssl/private

        # Default ciphers to use on SSL-enabled listening sockets.
        # For more information, see ciphers(1SSL). This list is from:
        #  https://hynek.me/articles/hardening-your-web-servers-ssl-ciphers/
        # An alternative list with additional directives can be obtained from
        #  https://mozilla.github.io/server-side-tls/ssl-config-generator/?server=haproxy
        ssl-default-bind-ciphers ECDH+AESGCM:DH+AESGCM:ECDH+AES256:DH+AES256:ECDH+AES128:DH+AES:RSA+AESGCM:RSA+AES:!aNULL:!MD5:!DSS
        ssl-default-bind-options no-sslv3

defaults
        log     global
        mode    http
        option  httplog
        option  dontlognull
        timeout connect 5000
        timeout client  50000
        timeout server  50000
        errorfile 400 /etc/haproxy/errors/400.http
        errorfile 403 /etc/haproxy/errors/403.http
        errorfile 408 /etc/haproxy/errors/408.http
        errorfile 500 /etc/haproxy/errors/500.http
        errorfile 502 /etc/haproxy/errors/502.http
        errorfile 503 /etc/haproxy/errors/503.http
        errorfile 504 /etc/haproxy/errors/504.http

frontend Local_Server
    bind *:80
    mode http
    default_backend k8s_server

backend k8s_server
    mode http
    balance roundrobin
    server web1.example.com  100.0.0.2:8080
```

You can run the following command to check the correctness of the configuration .
```
$ haproxy -c -f /etc/haproxy/haproxy.cfg
```
If your configuration is correct then you should see the following message

```
Configuration file is valid
```
Now you need to restart haproxy
```
$ sudo service haproxy restart
```

## 4. Setup kubernetes Ingress controller

Ingress controller are not set up by default inside the kubernetes cluster. We need to set up them manually. There are many Ingress controllers available such as -AKS Application Gateway Ingress Controller , Ambassador, AppsCode Inc., AWS ALB Ingress Controller, Contour, Citrix Ingress Controller, F5 BIG-IP, Gloo, Istio, Kong, Skipper, Traefik

But in our case we are going with NGINX Ingress Controller for Kubernetes.
If you like than there is official guide for setting up NGINX Ingress controller.
```https://docs.nginx.com/nginx-ingress-controller/installation/installation-with-manifests/```

### 1. You need to clone the git repo -
```
$ git clone https://github.com/nginxinc/kubernetes-ingress.git
```
### 2. Go to the directory kubernetes-ingress/deployments
```
cd kubernetes-ingress/deployments
```
### 3 Inside the deployments directory you will find namespace and service account yaml .e.g. ns-and-sa.yaml. Using this yaml we need to create namespace and service account for the Ingress controller.

You can find ns-and-sa.yaml, inside the directory common/ns-and-sa.yaml

```
$kubectl apply -f common/ns-and-sa.yaml

namespace/nginx-ingress created
serviceaccount/nginx-ingress created
```

### 4. As a next step you need to create cluster role and cluster role binding for the service account which we have created in step no 3.
For the cluster role and cluster role binding you can find rbac.yaml inside the directory rbac/rbac.yaml
```
$ kubectl apply -f rbac/rbac.yaml
```
```
clusterrole.rbac.authorization.k8s.io/nginx-ingress created
clusterrolebinding.rbac.authorization.k8s.io/nginx-ingress created
```

### 5. For the App protect role create the following role binding
```
kubectl apply -f rbac/ap-rbac.yaml
```

### 6. Now you need to create secret using TLS certificate and key for the server.
Use the ```default-server-secret.yaml``` available inside the directory ```common/default-server-secret.yaml```
```
$ kubectl apply -f common/default-server-secret.yaml
```
secret/default-server-secret created

### 7. For customizing NGINX configuration you need to create config map using nginx-config.yaml available at common/nginx-config.yaml
$ kubectl apply -f common/nginx-config.yaml
```
configmap/nginx-config created
````
### 8. Lets create ingress controller pod using the deployment
```
$ kubectl apply -f deployment/nginx-ingress.yaml
```
deployment.apps/nginx-ingress created

### 9. Now run it as a Daemon set
```
$ kubectl apply -f daemon-set/nginx-ingress.yaml
```
daemonset.apps/nginx-ingress

### 7. Now we can check all the container images running inside the namespace - nginx-ingress
```
$ kubectl get all -n nginx-ingress
```
After running the above command you should see something similar in your terminal
```
NAME                      READY   STATUS    RESTARTS   AGE
pod/nginx-ingress-hqghc   1/1     Running   0          42s
pod/nginx-ingress-jcxjv   1/1     Running   0          42s
BASH
NAME                           DESIRED   CURRENT   READY   UP-TO-DATE   AVAILABLE   NODE SELECTOR   AGE
daemonset.apps/nginx-ingress   2         2         2       2            2           <none>          42s
```
Till now we have setup NGINX controller but not the Ingress resource yet.

Before setting up Ingress resource first we need to deploy some application inside our kubernetes cluster.

## 5.Deploy spring boot microservice inside kubernetes cluster
Alright lets deploy spring boot microservice using follow command.
```
$ kubectl create deployment demo --image=rahulwagh17/kubernetes:jhooq-k8s-springboot
```
Check the deployment

```
$ kubectl get deployments
NAME   READY   UP-TO-DATE   AVAILABLE   AGE
demo   1/1     1            1           5h20m
```

Expose the deployment as service
```
$ kubectl expose deployment demo --type=ClusterIP --name=demo-service  --port=8080
```
service/demo-service exposed
Okay so now we have created the deployment and exposed the service as ClusterIP running on port 8080.

You can view the exposed service
```
$ kubectl get service demo-service
```
```
NAME           TYPE        CLUSTER-IP     EXTERNAL-IP   PORT(S)    AGE
demo-service   ClusterIP   10.233.62.13   <none>        8080/TCP   43h
```

## 6. Create Ingress resource
Before you create the Ingress resource you should be mindful about the two things -

```
deployed service name .e.g. - demo-service
service port .e.g. 8080
```
Now you should create a yaml with the name ingress-resource.yaml

```
$ touch ingress-resource.yaml
```
```
apiVersion: extensions/v1beta1
kind: Ingress
metadata:
  name: springboot-ingress
  annotations:
    ingress.kubernetes.io/rewrite-target: /
spec:
  rules:
  - host: jhooq.demo
    http:
      paths:
      - backend:
          serviceName: demo-service
          servicePort: 8080
```

Alright now after updating the ingress-resource.yaml, you need to create the ingress resource using following command
```
$ kubectl create -f ingress-resource.yaml
```

Once your Ingress resource is created, you can check it with the following command
```
$ kubectl describe ing springboot-ingress
```

You should be able to see something similar in your terminal
```
 Name:             springboot-ingress
Namespace:        default
Address:
Default backend:  default-http-backend:80 (<error: endpoints "default-http-backend" not found>)
Rules:
  Host        Path  Backends
  ----        ----  --------
  jhooq.demo
                 demo-service:8080 (10.233.90.4:8080)
Annotations:  ingress.kubernetes.io/rewrite-target: /
Events:
  Type    Reason          Age   From                      Message
  ----    ------          ----  ----                      -------
  Normal  AddedOrUpdated  103s  nginx-ingress-controller  Configuration for default/springboot-ingress was added or updated
  Normal  AddedOrUpdated  103s  nginx-ingress-controller  Configuration for default/springboot-ingress was added or updated
```

## 7. Use HAProxy to access the deployed microservice withing kubernetes cluster
Once you have implemented all the 6 steps then you are pretty much done setting up everything.

One last thing which remaining is to add an host entry into your /etc/hosts file.

Since our host IP address in this example is 100.0.0.2, so make an following entry inside the /etc/hosts

```
$ vi /etc/hosts
```
```
100.0.0.2 jhooq.demo
```
Lets test our microservice using the hostname .i.e. jhooq.demo

$ curl jhooq.demo/hello

Hello - Jhooq-k8s



