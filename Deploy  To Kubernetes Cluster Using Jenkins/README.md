
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
