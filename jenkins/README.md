
# Install Jenkins on Kubernetes

## Create a jenkins Docker image and upload to Docker hub

First we’ll create our Dockerfile with the command
```
sudo vi Dockfile
```

In that file, paste the following:

```
FROM jenkins/jenkins:alpine

# Distributed Builds plugins
RUN /usr/local/bin/install-plugins.sh ssh-slaves

#Install GIT
RUN /usr/local/bin/install-plugins.sh git

# Artifacts
RUN /usr/local/bin/install-plugins.sh htmlpublisher

# UI
RUN /usr/local/bin/install-plugins.sh greenballs
RUN /usr/local/bin/install-plugins.sh simple-theme-plugin

# Scaling
RUN /usr/local/bin/install-plugins.sh kubernetes

VOLUME /var/jenkins_home

USER jenkins
```
Save and close the file.

 build your image and push to dockerhub
 
 ```
 docker build -t nikhilnambiars/jenkins:latest .
 ```
 ```
 docker push nikhilnambiars/jenkins:latest
 ```
 
 ## Installing Jenkins on Kubernetes
 
 Create and open a new file called deployment.yaml using nano or your preferred editor:
```
nano deloyment.yaml
```

Now add the following code to define the Jenkins image, its port, and several more configurations:

```
apiVersion: apps/v1
kind: Deployment
metadata:
  name: jenkins-master-deployment
  labels: 
    app: jenkins-master
    version: latest
    group: jenkins
spec:
  replicas: 1
  strategy:
    type: Recreate
  selector:
      matchLabels:
        app: jenkins-master
        version: latest
        group: jenkins
  template:
    metadata:
      labels:
        app: jenkins-master
        version: latest
        group: jenkins
    spec:
      containers:
        - name: jenkins-master
          image: nikhilnambiars/jenkins:latest
          imagePullPolicy: IfNotPresent
          env:
          - name: JAVA_OPTS
            value: -Djenkins.install.runSetupWizard=false
          ports:
          - name: http-port
            containerPort: 8080
          - name: jnlp-port
            containerPort: 50000
          volumeMounts:
            - name: jenkins-home
              mountPath: /var/jenkins
      volumes:
      - name: jenkins-home
        persistentVolumeClaim:
          claimName: jenkins-pv-claim


---              
apiVersion: v1
kind: Service
metadata:
  labels:
    app: jenkins-master
    version: latest
    group: jenkins
  name: jenkins-master-service


spec:

  selector:
    app: jenkins-master
    version: latest
    group: jenkins


  ports:
  - protocol: "TCP"
    port: 8080
    targetPort: 8080
  type: NodePort      
  
  ---              
apiVersion: v1
kind: Service
metadata:
  labels:
    app: jenkins-master
    version: latest
    group: jenkins
  name: jenkins-jnlp


spec:

  selector:
    app: jenkins-master
    version: latest
    group: jenkins


  ports:
  - protocol: "TCP"
    port: 50000
    targetPort: 5000
  type: ClusterIP    
  
  ```
  This YAML file creates a deployment using the Jenkins LTS image and also opens port 8080 and 8080
  
  Now create this deployment
  
  ```
  kubectl create -f  deployment.yml
  ```
  Create the persistent volume and persistent volume claim by using following command
  
  ```
  kubectl create -f pv-pvc.yml
  ```
  
  
  Use kubectl to verify the pod’s state:
```
   kubectl get pods
```

Once the pod is running, you need to expose it using a Service. You will use the NodePort Service type for this tutorial.

In the above YAML file, you define your NodePort Service and then expose port 8080

Check that the Service is running:
```
kubectl get services 
```


in this tutorial jenkins start without admin password. because in env field we set as ```Djenkins.install.runSetupWizard=false```

if it was not set Let’s use kubectl to pull the password from those logs.

First, return to your terminal and retrieve your Pod name:
```
kubectl get pods
```

Next, check the Pod’s logs for the admin password

```
kubectl logs <pode name>
```

You might need to scroll up or down to find the password:

```

Running from: /usr/share/jenkins/jenkins.war
webroot: EnvVars.masterEnvVars.get("JENKINS_HOME")
. . .

Jenkins initial setup is required. An admin user has been created and a password generated.
Please use the following password to proceed to installation:

your_jenkins_password

This may also be found at: /var/jenkins_home/secrets/initialAdminPassword
```

Copy your_jenkins_password. Now return to your browser and paste it into the Jenkins UI.


