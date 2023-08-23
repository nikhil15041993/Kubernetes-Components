## Pull the Docker Image from AWS ECR in Kubernetes

Normally when we want to pull the images from AWS ECR to our localhost, we need to log in using the following command to gain access.

```
aws ecr get-login-password --region region | docker login --username AWS --password-stdin aws_account_id.dkr.ecr.region.amazonaws.com
```

Suppose you have gained access to AWS CLI, run the following command to get the login password to ECR registry
```
aws ecr get-login-password --region region
```

According to the official documentation from Kubernetes, we have to create new secret which contains the data called ```.dockerconfigjson```

For example, to support your docker images on Amazon ECR, the quickest way is running the command below with your AWS credentials
```
kubectl create secret docker-registry regcred --docker-server=<your-registry-server> --docker-username=<your-name> --docker-password=<your-pword> --docker-email=<your-email>
```

    * <your-registry-server> would be aws_account_id.dkr.ecr.region.amazonaws.com
    * <your-name> would be AWS
    * <your-pword> would be the login password from the AWS ECR command above
    * <your-email> would be the email of the AWS account


Once it is done, run the inspect command to check the generated secret
```
kubectl get secret regcred --output=yaml
```
and it would be similar to the format below.
```
apiVersion: v1
data:
  .dockerconfigjson: exhsjdfslfisdf89s7df9fs87f6dsfsf65...
kind: Secret
metadata:
  ...
  name: regcred
  ...
type: kubernetes.io/dockerconfigjson
```

Finally, under your deployment YAML file, define the secret name within imagePullSecrets
```
apiVersion: apps/v1
kind: Deployment
metadata:
  name: app-name
spec:
  replicas: 2
  selector:
    matchLabels:
      app: app-name
  template:
    metadata:
      labels:
        app: app-name
    spec:
      containers:
        - name: app
          image: aws_account_id.dkr.ecr.region.amazonaws.com/app_and_version
          …
      imagePullSecrets:
        - name: regcred
```
And your pods should be able to pull the images from AWS ECR and deploy properly.
