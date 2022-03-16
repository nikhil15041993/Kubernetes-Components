# Add Kubernetes Users to Your Cluster

For clusters created using other deployment tools, you may need to modify the instructions. While there are several ways to authenticate users, Here Iam using the X509 certificate approach in this blog.

## 1. Install OpenSSL

Logi to the master node and Install SSH on master node.

First, check if OpenSSL is installed:
```
openssl version
```
If OpenSSL is installed you will get a response showing the version. Otherwise, you will need to install OpenSSL:
```
sudo apt-get update

sudo apt-get install openssl
```

## 2. Add a new Kubernetes user account

I will create a new user named nikhil, and I will put that user in a group named dev.
create a directory in my file system and do my work in that directory.

```
mkdir kube-users
cd kube-users
```

### 3. Create and sign a certificate for the new user

First, create a private key:
```
openssl  genrsa -out nikhil.key 2048
```

Next, create a certificate signing request. The value of CN is the user name, and the value of O is the group name:

```
openssl req -new -key user-2.key -out nikhil.csr -subj "/CN=nikhil-2/O=dev"
```

Finally, sign the CSR to create a certificate signed by the Kubernetes CA. The resulting certificate in this example will not expire for 720 days.
```
sudo openssl x509 -req -CA /etc/kubernetes/pki/ca.crt -CAkey /etc/kubernetes/pki/ca.key -CAcreateserial -days 720 -in nikhil.csr -out nikhil.crt
```

## 4.Add the user to your kubeconfig

Create the new user’s credentials
I’ll use the approach of embedding the certificate data in the kubeconfig file so that it will still be available if I copy the file to another location. I must use an absolute path to the user’s crt and key files because a relative path such as ~/kube-users/user-2.crt will cause an error. If you are following along, you’ll need to change my-login to the name of your account.

```
kubectl config set-credentials nikhil --client-certificate='/home/nikhils/kue-users/nikhil.crt' --client-key='/home/nikhils/kube-users/nikhil.key' --embed-certs=true
```
