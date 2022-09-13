# Prometheus, Grafana dashboard for Kubernetes monitoring

## Install Helm chart

The next tool we need is Helm Chart. If you are working on setting up Prometheus and Grafan for the Kubernetes cluster then it is always recommended using the Helm Chart Repository - https://prometheus-community.github. io/helm-charts so that you do not miss any configuration step.
Helm Chart works out of the box and it will take care of everything for you by installing prometheus-alertnamanger, prometheus-server, prometheus-operator.

you can Install Helm Chart - Use the following script to install the helm chart -
```
curl -fsSL -o get_helm.sh https://raw.githubusercontent.com/helm/helm/master/scripts/get-helm-3
chmod 700 get_helm.sh
./get_helm.sh
```

## Install Kubernetes Metrics Server
Alright the next step would be to install the Kubernetes Metrics server onto the Kubernetes cluster so that Prometheus can collect the performance metrics of Kubernetes.

Kubernetes Metrics server collects the resource metrics from kubelets and exposes it to the Kubernetes api server through Metrics API for use by Horizontal Pod Autoscaler and Vertical Pod Autoscaler.

Use the following installation script to install kubernetes metrics server
```
kubectl apply -f https://github.com/kubernetes-sigs/metrics-server/releases/latest/download/components.yaml
```

Verify the Kubernetes metric server installation- You can verify the Kubernetes metric server installation ``` kubectl get deployment metrics-server -n kube-system``` or ```kubectl get pods -n kube-system```


## Install Prometheus
Now we have set up the Kubernetes cluster in the previous step, let's install the prometheus using the helm chart.

Add Prometheus helm chart repository
```
helm repo add prometheus-community https://prometheus-community.github.io/helm-charts 
```
Update the helm chart repository
```
helm repo update 
```

Create prometheus namespace
```
kubectl create namespace prometheus
```
Install the prometheus
```
helm install prometheus prometheus-community/prometheus \
    --namespace prometheus \
    --set alertmanager.persistentVolume.storageClass="gp2" \
    --set server.persistentVolume.storageClass="gp2" 
```
to view the all pods and service on prometheus namespace use ``` kubectl get all -n prometheus ```


View the Prometheus dashboard by forwarding the deployment ports
```
kubectl port-forward deployment/prometheus-server 9090:9090 -n prometheus
```

## Install grafana
After installing Prometheus let's install the Grafana using the Helm Chart.

Add the Grafana helm chart repository
```
helm repo add grafana https://grafana.github.io/helm-charts 
```

Update the helm chart repository
```
helm repo update 
```

Now we need to create a Prometheus data source so that Grafana can access the Kubernetes metrics. Create a yaml file prometheus-datasource.yaml and save the following data source configuration into it -

```
datasources:
  datasources.yaml:
    apiVersion: 1
    datasources:
    - name: Prometheus
      type: prometheus
      url: http://prometheus-server.prometheus.svc.cluster.local
      access: proxy
      isDefault: true
```


Create a namespace grafana
```
kubectl create namespace grafana
```
Install the Grafana

```
helm install grafana grafana/grafana \
    --namespace grafana \
    --set persistence.storageClassName="gp2" \
    --set persistence.enabled=true \
    --set adminPassword='EKS!sAWSome' \
    --values prometheus-datasource.yaml \
    --set service.type=LoadBalancer 
```
Verify the Grafana installation by using the following kubectl command -
```
kubectl get all -n grafana
```
to Access the Grafana dashboard we need to find Public IP address and for that use the following command -
```
kubectl get service -n grafana 
```
Copy the Public IP address and open it in the browser -


## Deploy a spring boot microservice and monitor it on Grafana
To deploy the spring boot on kubernetes cluster user the following kubectl command
```
apiVersion: apps/v1
kind: Deployment
metadata:
  name: jhooq-springboot
spec:
  replicas: 2
  selector:
    matchLabels:
      app: jhooq-springboot
  template:
    metadata:
      labels:
        app: jhooq-springboot
    spec:
      containers:
        - name: springboot
          image: rahulwagh17/kubernetes:jhooq-k8s-springboot

          resources:
            requests:
              memory: "128Mi"
              cpu: "512m"
            limits:
              memory: "128Mi"
              cpu: "512m"

          ports:
            - containerPort: 8080

          readinessProbe:
            httpGet:
              path: /hello
              port: 8080
            initialDelaySeconds: 15
            periodSeconds: 10

          livenessProbe:
            httpGet:
              path: /hello
              port: 8080
            initialDelaySeconds: 15
            periodSeconds: 10

          startupProbe:
            httpGet:
              path: /hello
              port: 8080
            failureThreshold: 30
            periodSeconds: 10

          env:
            - name: PORT
              value: "8080"
---
apiVersion: v1
kind: Service
metadata:
  name: jhooq-springboot
spec:
  type: NodePort
  ports:
    - port: 80
      targetPort: 8080
  selector:
    app: jhooq-springboot
```

Then create the application by 
```
kubectl apply -f k8s-spring-boot-deployment.yml 
```

## Verify the deployment by running the following kubectl command
```
kubectl get deployment jhooq-springboot
```
```
NAME               READY   UP-TO-DATE   AVAILABLE   AGE
jhooq-springboot   2/2     2            2           8m5s
```

Refresh the grafana dashboard to verify the deployment
