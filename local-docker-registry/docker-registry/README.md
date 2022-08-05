
## create the pode using docker private registry

```
kubectl run mytest --image 10.0.0.1:5000/myimage:vi --image-pull-policy Always
```


## Generating certificate/key pair for your private Docker registry
```
mkdir certs
```
Copy below content to __openssl.conf__

_Update **Docker Server IP** with the IP address of your server where you will be running docker registry_
```
[ req ]
distinguished_name = req_distinguished_name
x509_extensions     = req_ext
default_md         = sha256
prompt             = no
encrypt_key        = no

[ req_distinguished_name ]
countryName            = "GB"
localityName           = "London"
organizationName       = "Just Me and Opensource"
organizationalUnitName = "YouTube"
commonName             = "<Docker Server IP>"
emailAddress           = "test@example.com"

[ req_ext ]
subjectAltName = @alt_names

[alt_names]
DNS = "<Docker Server IP>"
```
Generate the certificate and private key
```
openssl req \
 -x509 -newkey rsa:4096 -days 365 -config openssl.conf \
 -keyout certs/domain.key -out certs/domain.crt
```
To verify your certificate
```
openssl x509 -text -noout -in certs/domain.crt
```

## Auth base configuration

if we choose auth base configuration
install apache on master node and check the htpasswd

```
which hdpasswd
```

```
mkdir auth
```
## create a user to access the directory

```
htpasswd -Bbn testuser testpassword > auth/hdpasswd
```

next create the registry using yaml file...

if you want to push/pull the image first login to the docker using htpasswd

```
docker login
```

### Run this image to kubernets pods

for test the events on background

```
kubectl get events -w 
````

### create a secret

```
kubectl create secret docker-registry mydockercred --docker-server 10.0.0.1:5000 --docker-username testuser --docker-password testpassword
```
run a dry run for create the yaml file dry run help to reate the yaml file automatically

```
kubectl run mytest --image 10.0.0.1:5000/myimage:vi --image-pull-policy Always > /tmp/mytest.yml
```

now you can edit this  myimage.yml file


in spec potion under containers add ``` ImagePullSecrets -name: mydockercred ```




