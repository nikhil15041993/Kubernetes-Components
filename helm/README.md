# kubernets-Helm-Chart

## Helm Chart

Helm Chart is an application package manager for the kubernetes cluster, just like we have apt packager manager in Linux.

So what can we do with Helm Chart -

* Define k8s application
* Install k8s application
* Upgrade k8s application

Most important aspect of the Helm Chart is you do not have to use Kubernetes CLI(command line interface) and neither you need to remember complex kubernetes commands to manage the kubernetes manifest.

## 1 Install Helm Chart

Using Script

```
curl -fsSL -o get_helm.sh https://raw.githubusercontent.com/helm/helm/master/scripts/get-helm-3
```
```
chmod 700 get_helm.sh
```
```
./get_helm.sh
```
Verify Helm Chart Installation
```
which helm
```

## 2 create Our First Helm Chart

Before we create a our First Helm Chart, we need to have kubernetes cluster up and running. Use the following command to verify the status of kubernetes cluster -
```
kubectl get all
```
```
NAME                 TYPE        CLUSTER-IP   EXTERNAL-IP   PORT(S)   AGE
service/kubernetes   ClusterIP   10.233.0.1   <none>        443/TCP   27d
```

lets run some Helm Commands for creating our first Helm Chart
```
helm create springboot
```

## 3. Helm Chart Structure

Run the following command to see the tree structure of our Springboot Helm Chart -

```
tree springboot
```
```
springboot
├── Chart.yaml
├── charts
├── templates
│ ├── NOTES.txt
│ ├── _helpers.tpl
│ ├── deployment.yaml
│ ├── hpa.yaml
│ ├── ingress.yaml
│ ├── service.yaml
│ ├── serviceaccount.yaml
│ └── tests
│     └── test-connection.yaml
└── values.yaml
```
In the next section we will go through each YAML configuration

### Chart.yaml
This file contains all the metadata about our Helm Chart for example -

```
apiVersion: v2 #mandatory
name: springboot #mandatory
description: A Helm chart for Kubernetes
type: application
version: 0.1.0 #mandatory
appVersion: 1.16.0
```
1 apiVersion
2 name
3 version
Other configurations are optional

Versioning - Each chart should has its own version number and it should follow the Semantic Versioning

### values.yaml
 
This configuration file holds values for the configuration.

```
replicaCount: 1
image:
  repository: rahulwagh17/kubernetes:jhooq-k8s-springboot #updated url
  pullPolicy: IfNotPresent
  tag: ""
imagePullSecrets: []
nameOverride: ""
fullnameOverride: ""
serviceAccount:
  create: true
  annotations: {}
  name: ""
podAnnotations: {}
podSecurityContext: {}

securityContext: {}
service:
  type: ClusterIP
  port: 8080  #updated port
ingress:
  enabled: false
  annotations: {}
  hosts:
    - host: chart-example.local
      paths: []
  tls: []
resources: {}

autoscaling:
  enabled: false
  minReplicas: 1
  maxReplicas: 100
  targetCPUUtilizationPercentage: 80

nodeSelector: {}

tolerations: []

affinity: {}
```

lets go through the configuration which we need to modify for deploying our spring boot application.

repository : image:repository: rahulwagh17/kubernetes:jhooq-k8s-springboot
port: 8080

So in the whole values.yaml you need to update the configuration at two places repository and port

### deployment.yaml

The next configuration we need to update is deployment.yaml and as the name suggest it is used for deployment purpose.

```
apiVersion: apps/v1
kind: Deployment
metadata:
  name: {{ include "springboot.fullname" . }}
  labels:
    {{- include "springboot.labels" . | nindent 4 }}
spec:
{{- if not .Values.autoscaling.enabled }}
  replicas: {{ .Values.replicaCount }}
{{- end }}
  selector:
    matchLabels:
      {{- include "springboot.selectorLabels" . | nindent 6 }}
  template:
    metadata:
    {{- with .Values.podAnnotations }}
      annotations:
        {{- toYaml . | nindent 8 }}
    {{- end }}
      labels:
        {{- include "springboot.selectorLabels" . | nindent 8 }}
    spec:
      {{- with .Values.imagePullSecrets }}
      imagePullSecrets:
        {{- toYaml . | nindent 8 }}
      {{- end }}
      serviceAccountName: {{ include "springboot.serviceAccountName" . }}
      securityContext:
        {{- toYaml .Values.podSecurityContext | nindent 8 }}
      containers:
        - name: {{ .Chart.Name }}
          securityContext:
            {{- toYaml .Values.securityContext | nindent 12 }}
          image: "{{ .Values.image.repository }}"   #update here
          imagePullPolicy: {{ .Values.image.pullPolicy }}
          ports:
            - name: http
              containerPort: 8080     #update here
              protocol: TCP           #comment it
              #livenessProbe:           #comment it
              #httpGet:           #comment it
              #path: /           #comment it
              #port: http           #comment it
              #readinessProbe:           #comment it
              #httpGet:           #comment it
              #path: /           #comment it
              #port: http           #comment it
          resources:
            {{- toYaml .Values.resources | nindent 12  }}
      {{- with .Values.nodeSelector }}
      nodeSelector:
        {{- toYaml . | nindent 8 }}
      {{- end }}
      {{- with .Values.affinity }}
      affinity:
        {{- toYaml . | nindent 8 }}
      {{- end }}
      {{- with .Values.tolerations }}
      tolerations:
        {{- toYaml . | nindent 8 }}
      {{- end }}
```
we need to update the containerPort to 8080

containerPort : 8080
Because we need to deploy our spring boot application at port 8080.

### service.yaml

The service.yaml is basically used for exposing our kubernetes springboot deployment as service. The good thing over here we do not need to update any configuration here.

```
apiVersion: v1
kind: Service
metadata:
  name: {{ include "springboot.fullname" . }}
  labels:
    {{- include "springboot.labels" . | nindent 4 }}
spec:
  type: {{ .Values.service.type }}
  ports:
    - port: {{ .Values.service.port }}
      targetPort: http
      protocol: TCP
      name: http
  selector:
    {{- include "springboot.selectorLabels" . | nindent 4 }}
```

## 4 Run  helm template springboot


```
helm template springboot
```
After running the above command it should return you will service.yaml, deployment.yaml and test-connection.yaml with actual values

There is one more sanitary command lint provided by helm which you could run to identify possible issues forehand.

```
helm lint springboot
```
```
==> Linting springboot
[INFO] Chart.yaml: icon is recommended

1 chart(s) linted, 0 chart(s) failed
```
## 5  helm install

Alright lets run the install command.
```
helm install myfirstspringboot springboot
```
There are two name which we used during the installation command -

myfirstspringboot : It’s a release name for helm chart otherwise helm will generate its own release name that is why we have assigned this name.
springboot : It is our actual chart name.


## 6. Verify the helm install

Use the following list command to list down all the releases -
```
helm list -a
```
Lets do some cross verification using kubectl commands also, so that we can make sure helm has done its work.
```
kubectl get all
```
Now we have installed our first helm chart of springboot application. Lets test the spring boot application by accessing it by ip

## 7 Upgrade helm release

There is one more feature of Helm Chart which is helm upgrade. It makes it easy for devops to release the new version of application.
Lets take the same myfirstspringboot release and update its replicaCount from 1 to 2

But before we need to update the replicaCount. First we need to udpate the version in Chart.yaml from 0.1.0 to 0.1.1

```
apiVersion: v2
name: springboot
description: A Helm chart for Kubernetes
type: application
version: 0.1.1
appVersion: 1.16.0
```
Alright now lets update the replicaCount in values.yaml

```
replicaCount: 2
image:
  repository: rahulwagh17/kubernetes:jhooq-k8s-springboot
  pullPolicy: IfNotPresent
  tag: ""
imagePullSecrets: []
nameOverride: ""
fullnameOverride: ""
serviceAccount:
  create: true
  annotations: {}
  name: ""
podAnnotations: {}
podSecurityContext: {}

securityContext: {}
service:
  type: ClusterIP
  port: 8080
ingress:
  enabled: false
  annotations: {}
  hosts:
    - host: chart-example.local
      paths: []
  tls: []
resources: {}

autoscaling:
  enabled: false
  minReplicas: 1
  maxReplicas: 100
  targetCPUUtilizationPercentage: 80

nodeSelector: {}

tolerations: []

affinity: {}
```

Okay now we are done with the update and ready to make our new release upgrade. Use the following command -
```
helm upgrade myfirstspringboot .
```
You should see a message with your release name .i.e. - Release “myfirstspringboot” has been upgraded. Happy Helming!

Verify your helm upgrade by running following list command
```
helm list -a
```
As you can see now the revision count is 2.

Lets again run kubectl get deployment
```
kubectl get deployments
```
## 8 Rollback Helm release

 we upgraded the Helm chart release from version 1 to version 2.

So let’s see one more rollback feature of Helm Chart.
```
helm rollback myfirstspringboot 1
```
As you can see from the above screenshot we successfully rolled back the release to the previous version. But one interesting thing about Helm is, it still updates the REVISION to 3
```
helm list -a
```
To confirm that we have actually rolled back our Helm Chart release, lets run some kubectl commands
```
kubectl get deployments
```

## 8 Delete Helm release

Wouldn’t it be nice if you need to run only one command to delete your Helm release and you do not have to do anything else.

```
helm delete myfirstspringboot
```















https://jhooq.com/building-first-helm-chart-with-spring-boot/
