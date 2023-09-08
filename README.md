# Kubernetes-Components

NOTE: When deploying container in Proxmox, we need to comment the SWAP and OS high performance modules in the create_pushpin/roles/prepare/tasks/main.yml file.

We have to make sure that the OS version of our VM is 20.04 and Set swap size to "8G" also swappines 60.

This playbook is for setup pushpin.

Configure variables in all.yml


```
Examples:
domain_name: "statemine.public.curie.radiumblock.co" #domain name
upstream_domain_name: "statemine.backend.radiumblock.io" #upstream domain name
service_name: 'statemine' #service_name
swap_vars_size: "8G"   
swap_vars_swappiness: "60"

#route53 variables
zone_domain_name: "radiumblock.io" #Domain name of route53 zone
zone_subdomain_name: "statemine.curie-accelerator.radiumblock.io" #SubDomain name of route53

#sqs variables
aws_sqs_access_key: '' #aws region where sqs is present
aws_sqs_secret_key: '' #aws secret key for sqs
awsregionname: '' #aws access key for sqs
service_name: '' #service_name
```


Add the acquired VM IP addresses to the [servers:children] section of the inventory file to ensure the playbook operates in accordance with these IPs.


### Configure couples of variables in inventory file

1. For ```caller_reference_for_healthcheck variable```, we have to define any unique string.(For example, output of the Date command)

2. For ```cloudwath_alarm_name_Initial``` variable define name of cloudwatch alarm and after this variable still there is another suffix name that have already designed in yaml file. And that will Endpoint Node - Health Check so for example. (Example: Astar Cache Singapore)

3. if ```cloudwath_alarm_name_Initial``` name like "Polkadot N.virgina" then alarm name will be like this Polkadot N.virgina Endpoint Node - Health Check .

4. ```server_region``` which indicates that where our server is located.(This is used by sqs, eg. astar-cache-frankfurt)

5. ```route_region``` which indicates that for which region we will set latency for route53 domain. And that should be in aws region terminology. Ex. "us-east-1"

6. For ```health_check_name```, give for example Astar Cache Singapore


### configure SSL

To configure the SSL certificate for Nginx, generate an SSL certificate using the domain name defined in the 'group_vars/all.yml' file.

```
domain_name: "statemine.public.curie.radiumblock.co" #domain name
```

Place your SSL certs under this folder.
```
roles/nginx/templates

fullchain.pem will be certs file
privkey.pem will be key file
```

### Run ansible playbook in dry mode.

```
ansible-playbook -i inventory main.yml --check -v
```

**Run ansible playbook.**

```
ansible-playbook -i inventory main.yml -v
```




