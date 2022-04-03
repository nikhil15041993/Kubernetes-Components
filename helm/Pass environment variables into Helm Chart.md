# Pass environment variables into Helm Chart

Helm chart provides a couple of ways to access or pass environment variables into the templates

* set
* values.yaml

Create a helm chart name helloworld

```
 helm create helloworld
```

## 1. Pass environment variables using set

here we are going to set the replicatCount=2

```
helm install --set replicaCount=2 helloworld-1 world
```

This command will set the replicaCount to 2.

You can verify it using the following command -
```
kubectl get deployment
```
### Set multiple values and Overriding

--set also provides a way to use multiple values as well as you can override the values also.

```
helm install --set replicaCount=2  --set replicaCount=3 helloworldrelease helloworld
```

## 2.Pass environment variables using values.yaml

First, create myvalues.yaml

```
vi myvalues.yaml
```
Add following replica count entry to it
```
replicaCount: 2
```

Install the helm chart -
```
helm install -f myvalues.yaml helloworldrelease helloworld 
```

Verify -
```
kubectl get deployment
```


### Iterate over key-value map

Create a file named env-values.yaml and store the above key-value map inside it.
```
vi env-values.yaml

examplemap:
 - name: "USERNAME"
   value: "test1"
 - name: "PASSWORD"
   value: "test2"
   
 ```
 
 To iterate or loop over the map first we need to create a place holder inside the Helm template.

```
env:
    {{- range .Values.examplemap }}
    - name: {{ .name }}
      value: {{ .value }}
    {{- end }} 
```

This is how my deployment.yaml would look like after putting the placeholder -


```
      securityContext:
        {{- toYaml .Values.podSecurityContext | nindent 8 }}
      containers:
        - name: {{ .Chart.Name }}
          securityContext:
            {{- toYaml .Values.securityContext | nindent 12 }}
          image: "{{ .Values.image.repository }}:{{ .Values.image.tag | default .Chart.AppVersion }}"
          imagePullPolicy: {{ .Values.image.pullPolicy }}
          env:
            {{- range .Values.examplemap }}         --Iteration begins over here
            - name: {{ .name }}
              value: {{ .value }}
            {{- end }}                              --Iteration ends over here
```

Let’s verify the Helm chart with helm template command

```
helm template -f env-values.yaml helloworld 
```


Install the helm chart

```
helm install -f env-values.yaml helloworldrelease helloworld
```
