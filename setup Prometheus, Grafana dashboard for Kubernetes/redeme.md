## setup Prometheus, Grafana dashboard for Kubernetes

### 1. Install Helm chart

If you are working on setting up Prometheus and Grafan for the Kubernetes cluster then it is always recommended using the Helm Chart Repository - https://prometheus-community.github. io/helm-charts so that you do not miss any configuration step.

### Install Helm Chart - Use the following script to install the helm chart -

```
curl -fsSL -o get_helm.sh https://raw.githubusercontent.com/helm/helm/master/scripts/get-helm-3
chmod 700 get_helm.sh
./get_helm.sh
```
Verify Helm Chart installation
```
WARNING: Kubernetes configuration file is group-readable. This is insecure. Location: /home/vagrant/.kube/config
version.BuildInfo{Version:"v3.4.0", GitCommit:"7090a89efc8a18f3d8178bf47d2462450349a004", GitTreeState:"clean", GoVersion:"go1.14.10"}
```

### Install Kubernetes Metrics Server
Alright the next step would be to install the Kubernetes Metrics server onto the Kubernetes cluster so that Prometheus can collect the performance metrics of Kubernetes.

Kubernetes Metrics server collects the resource metrics from kubelets and exposes it to the Kubernetes api server through Metrics API for use by Horizontal Pod Autoscaler and Vertical Pod Autoscaler.

The Kubernetes metrics offers -

* Fast autoscaling, collecting metrics
* Scalable support up to 5,000 node clusters.
* Resource efficiency and it requires very less cpu(1 mili core) and less memory(2 mb)
* Installation of Kubernetes metrics server
 Use the following installation script to install kubernetes metrics server
```
kubectl apply -f https://github.com/kubernetes-sigs/metrics-server/releases/latest/download/components.yaml
```

```
Note - In case you face the issue error: exec plugin: invalid apiVersion "client.authentication.k8s.io/v1alpha1" while running the kubectl apply -f command then follow these troubleshooting steps
```


Verify the Kubernetes metric server installation- You can verify the Kubernetes metric server installation 
```
kubectl get deployment metrics-server -n kube-system or kubectl get pods -n kube-system
```

### Install Prometheus
Now we have set up the Kubernetes cluster in the previous step, let's install the prometheus using the helm chart.

Add Prometheus helm chart repository
```
helm repo add prometheus-community https://prometheus-community.github.io/helm-charts 

Update the helm chart repository
helm repo update
```


### Create prometheus namespace
```
kubectl create namespace prometheus
```

### Install the prometheus
```
helm install prometheus prometheus-community/prometheus \
    --namespace prometheus \
    --set alertmanager.persistentVolume.storageClass="gp2" \
    --set server.persistentVolume.storageClass="gp2"
```

### View the Prometheus dashboard by forwarding the deployment ports
```
kubectl port-forward deployment/prometheus-server 9090:9090 -n prometheus
```

### Install grafana
After installing Prometheus let's install the Grafana using the Helm Chart.

Add the Grafana helm chart repository
```
helm repo add grafana https://grafana.github.io/helm-charts
```

### Update the helm chart repository
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
### Install the Grafana
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



To Access the Grafana dashboard we need to find Public AWS IP address and for that use the following command -
```
kubectl get service -n grafana
```

Copy the Public AWS IP address and open it in the browser -


### Import Grafana dashboard from Grafana Labs
Now we have set up everything in terms of Prometheus and Grafana. For the custom Grafana Dashboard, we are going to use the open source grafana dashboard.

On opensource grafana dashboard you can find many opensource dashboards which can be imported directly into the Grafana. For this session, I am going to import a dashboard 6417

How to import Grafana Dashboard

On left navigation click on + sign


Then goto import and enter the grafana dashboard number 6417


Select the data source and save it.


Now you can see your Grafana dashboard displaying all the Kubernetes metrics.
