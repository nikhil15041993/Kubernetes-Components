
# Deploying to Kubernetes cluster using Jenkins

### Pre-requistes:

1. Amazon EKS Cluster is setup and running. Click here to learn how to create Amazon EKS cluster.
2. Jenkins Master is up and running
3. Setup Jenkins slave, install docker in it.
4. Docker, Docker pipeline and Kubernetes Continuous Deploy plug-ins are installed in Jenkins
6. Install kubectl on your instance


## Create Credentials for Kubernetes Cluster

Click on Add Credentials, use Kubernetes configuration from drop down.



![add](https://user-images.githubusercontent.com/87381349/163588220-5ea8890c-2986-433a-9412-3f8a73393423.png)
![K8S_ID](https://user-images.githubusercontent.com/87381349/163588236-b5134ddd-809c-4a8c-8e7f-36d6362d630f.png)


execute the below command to get kubeconfig info, copy the entire content of the file:
```
sudo cat ~/.kube/config
```
Enter ID as K8S and choose enter directly and paste the above file content and save.

## set a clusterrole as cluster-admin

By default, your clusterrolebinding has system:anonymous set which blocks the cluster access.

Execute the following command, it will set a clusterrole as cluster-admin which will give you the required access.
```
kubectl create clusterrolebinding cluster-system-anonymous --clusterrole=cluster-admin --user=system:anonymous
```

## Create a pipeline in Jenkins

##  Copy the pipeline code from below

```

pipeline {

  agent any

  stages {

    stage('Checkout Source') {
      steps {
        checkout([$class: 'GitSCM',
         branches: [[name: '*/main']],
          extensions: [],
           userRemoteConfigs: [[credentialsId: 'git-lab-ssh', 
           url: 'git@gitlab.com:contact.nikhilnambiar/deploying-to-kubernetes-cluster-using-jenkins.git']]])
      }
    }



    stage('Deploy App') {
        kubernetesDeploy(
                    configs: '<path config yml file>',
                    kubeconfigId: '<kube credential>',
                    enableConfigSubstitution: true
                    )             
    }

    

    

  }

}

```

## Build the pipeline
Once you create the pipeline and changes values per your Docker user id and credentials ID, click on 

## Verify deployments to K8S

```
kubectl get pods
```

```
kubectl get deployments
```

```
kubectl get services
```
