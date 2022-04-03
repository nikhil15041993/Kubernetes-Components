## 1. Add ‘/bitnami/wordpress’ wordpress repo

### Search for the ‘wordpress’ repo
First of all you need to check how many wordpress repositories are available on the Helm Hub.

Use the following command to search for the wordpress repositories.

```
helm search hub wordpress
```
After running the above command it should return you with the list of repos available on the Helm Hub.
```
URL                                               	CHART VERSION 	APP VERSION        	DESCRIPTION                                       
https://artifacthub.io/packages/helm/kube-wordp...	0.1.0         	1.1                	this is my wordpress package                      
https://artifacthub.io/packages/helm/bitnami/wo...	13.1.12       	5.9.2              	WordPress is the world's most popular blogging ...
https://artifacthub.io/packages/helm/bitnami-ak...	13.1.11       	5.9.2              	WordPress is the world's most popular blogging ...
https://artifacthub.io/packages/helm/riftbit/wo...	12.1.16       	5.8.1              	Web publishing platform for building blogs and ...
https://artifacthub.io/packages/helm/sikalabs/w...	0.2.0         	                   	Simple Wordpress                                  
https://artifacthub.io/packages/helm/camptocamp...	0.6.10        	4.8.1              	Web publishing platform for building blogs and ...
https://artifacthub.io/packages/helm/groundhog2...	0.5.3         	5.9.2-apache       	A Helm chart for Wordpress on Kubernetes          
https://artifacthub.io/packages/helm/mcouliba/w...	0.1.0         	1.16.0             	A Helm chart for Kubernetes                       
https://artifacthub.io/packages/helm/homeenterp...	0.4.0         	5.8.0-php8.0-apache	Blog server                                       
https://artifacthub.io/packages/helm/wordpress-...	1.0.0         	1.1                	This is a package for configuring wordpress and...           

```

we are interested in https://hub.helm.sh/charts/bitnami/wordpress.
In case if the URL is too long to see then you can put --max-col-width=0, so that you can view the complete URL
```
helm search hub wordpress  --max-col-width=0
```
### Add ‘bitnami/wordpress’ to your repo list of Helm Chart

But before adding the bitnami/wordpress first check whether it already exists on your repo list or not?

```
helm repo list
```

If you haven added the bitnami/wordpress  then it should  show in the list.

let us add it to your repo list -
```
helm repo add bitnami https://charts.bitnami.com/bitnami
```
Once you add it successfully then you should see the following message.
```
"bitnami" has been added to your repositories
```

### Check Wordpress Version

Before we go into the installation step of the chart, let’s check the version of the WordPress which we are going to install.

Run the following command to get all the available versions -
```
helm search repo wordpress --versions
```

It will return a  list of all the version available for WordPress

```
NAME                   	CHART VERSION	APP VERSION   	DESCRIPTION                                       
bitnami/wordpress      	13.1.4       	5.9.2         	WordPress is the world's most popular blogging ...
bitnami/wordpress      	13.1.3       	5.9.2         	WordPress is the world's most popular blogging ...
bitnami/wordpress      	13.1.2       	5.9.2         	WordPress is the world's most popular blogging ...
bitnami/wordpress      	13.1.1       	5.9.2         	WordPress is the world's most popular blogging ...
bitnami/wordpress      	13.1.0       	5.9.2         	WordPress is the world's most popular blogging ...
bitnami/wordpress      	13.0.23      	5.9.2         	WordPress is the world's most popular blogging ...
bitnami/wordpress      	13.0.22      	5.9.1         	WordPress is the world's most popular blogging ...
bitnami/wordpress      	13.0.21      	5.9.1         	WordPress is the world's most popular blogging ...
bitnami/wordpress      	13.0.20      	5.9.1         	WordPress is the world's most popular blogging ...
bitnami/wordpress      	13.0.19      	5.9.1         	WordPress is the world's most popular blogging ...
bitnami/wordpress      	13.0.18      	5.9.1         	WordPress is the world's most popular blogging ...
bitnami/wordpress      	13.0.17      	5.9.1         	WordPress is the world's most popular blogging ...
bitnami/wordpress      	13.0.16      	5.9.1         	WordPress is the world's most popular blogging ...
```

We will go with the latest version which is 13.1.4

If you are familiar with WordPress before then you need username and password to access the WordPress CMS, so you can view the default values
```
helm show values bitnami/wordpress --version 13.1.4
```

## 2. Setup User account along with Username and Password for WordPress

create a complete user account and store it in wordpress-values.yaml
Copy and paste the following values

```
wordpressUsername: test
wordpressPassword: test
wordpressEmail: test@gail.com
wordpressFirstName: Admin
wordpressLastName: Admin
wordpressBlogName: test.com
service: 
  type: LoadBalancer
```

Save and Exit the file


## 3. Install the WordPress helm chart

 Let’s start installing the WordPress helm chart
 
 ###  Create a namespace - nswordpress
 
 ```
 kubectl create namespace nswordpress
 ```
 ### Verify the namespace
 
 ```
 kubectl get namespace
 ```
 
 ### Install wordpress helm chart
 
 ```
 helm install wordpress bitnami/wordpress --values=wordpress-values.yaml --namespace nswordpress --version 13.1.4
 ```
 Once you execute this command then it should return you with the following output
 
 ```
 NAME: wordpress
LAST DEPLOYED: Mon Nov 23 19:39:36 2020
NAMESPACE: nswordpress
STATUS: deployed
REVISION: 1
NOTES:
** Please be patient while the chart is being deployed **

Your WordPress site can be accessed through the following DNS name from within your cluster:

    wordpress.nswordpress.svc.cluster.local (port 80)

To access your WordPress site from outside the cluster follow the steps below:

1. Get the WordPress URL by running these commands:

   export NODE_PORT=$(kubectl get --namespace nswordpress -o jsonpath="{.spec.ports[0].nodePort}" services wordpress)
   export NODE_IP=$(kubectl get nodes --namespace nswordpress -o jsonpath="{.items[0].status.addresses[0].address}")
   echo "WordPress URL: http://$NODE_IP:$NODE_PORT/"
   echo "WordPress Admin URL: http://$NODE_IP:$NODE_PORT/admin"

2. Open a browser and access WordPress using the obtained URL.

3. Login with the following credentials below to see your blog:

  echo Username: test
  echo Password: $(kubectl get secret --namespace nswordpress wordpress -o jsonpath="{.data.wordpress-password}" | base64 --decode)
```
 
 ### Access URL for WordPress
 
 command which you need to execute from the output is 
 
 ```
 export NODE_PORT=$(kubectl get --namespace nswordpress -o jsonpath="{.spec.ports[0].nodePort}" services wordpress)
   export NODE_IP=$(kubectl get nodes --namespace nswordpress -o jsonpath="{.items[0].status.addresses[0].address}")
   echo "WordPress URL: http://$NODE_IP:$NODE_PORT/"
   ```
   It will return you with URL with IP address
   
 ### Access URL for WordPress Admin Portal
 
 ```
 echo "WordPress Admin URL: http://$NODE_IP:$NODE_PORT/admin"
 ```
 It will return you with URL with IP address
 
 ### Check all the Kubernetes resources status
 
 ```
 watch -x kubectl get all --namespace nswordpress
```
