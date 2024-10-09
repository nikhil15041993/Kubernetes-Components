## Step 1: Provision the VMs using Vagrant

First we need to provision the VMs using vagrant

We will be setting up total 3 VMs (Virtual Machine) with its unique IP -

Ansible Node (amaster) - 100.0.0.1 - 2 CPU - 2 GB Memory
Kubernetes Master Node (kmaster) - 100.0.0.2 - 2 CPU - 2 GB Memory
Kubernetes Worker Node (kworker) - 100.0.0.3 - 2 CPU - 2 GB Memory


Here is the Vagratnfile

```
Vagrant.configure("2") do |config|
  config.vm.define "amaster" do |amaster|
    amaster.vm.box_download_insecure = true
    amaster.vm.box = "hashicorp/bionic64"
    amaster.vm.network "private_network", ip: "100.0.0.1"
    amaster.vm.hostname = "amaster"
    amaster.vm.provider "virtualbox" do |v|
      v.name = "amaster"
      v.memory = 2048
      v.cpus = 2
    end
  end

  config.vm.define "kmaster" do |kmaster|
    kmaster.vm.box_download_insecure = true
    kmaster.vm.box = "hashicorp/bionic64"
    kmaster.vm.network "private_network", ip: "100.0.0.2"
    kmaster.vm.hostname = "kmaster"
    kmaster.vm.provider "virtualbox" do |v|
      v.name = "kmaster"
      v.memory = 2048
      v.cpus = 2
    end
  end

  config.vm.define "kworker" do |kworker|
    kworker.vm.box_download_insecure = true
    kworker.vm.box = "hashicorp/bionic64"
    kworker.vm.network "private_network", ip: "100.0.0.3"
    kworker.vm.hostname = "kworker"
    kworker.vm.provider "virtualbox" do |v|
      v.name = "kworker"
      v.memory = 2048
      v.cpus = 2
    end
  end

end

```

Start the vagrant box -

```
vagrant up
```


## Step 2: Update /etc/hosts on all the nodes

After starting the vagrant box you need to update the /etc/hosts file on each node .i.e -amaster, kmaster, kworker

So run the following command on all the three nodes
```
sudo vi /etc/hosts
```
Add the following entries in the hosts files of each node (amaster, kmaster, kworker)

```
100.0.0.1 amaster.com amaster
100.0.0.2 kmaster.com kmaster
100.0.0.3 kworker.com kworker
```


## Step 3: Generate SSH key for ansible (only need to run on ansible node .i.e. amaster)

To setup the kubespray smoothly we need to generate the SSH keys for the ansible master(amaster) nodes and copy the ssh keys to other nodes. 

Generate SSH key

```
ssh-keygen -t rsa
```


## Step 4: Copy SSH key to other nodes .i.e. - kmaster, kworker

Copy to kmaster node (During the ssh-copy-id it will ask for the other node password, so in case if you have not set any password then you can supply default password .i.e. vagrant) -

```
ssh-copy-id 100.0.0.2
ssh-copy-id 100.0.0.3
```

## Step 5: Install python3-pip (only need to run on ansible node .i.e. amaster)

Before installing the python3-pip, you need to download and update the package list from the repository.

```
sudo apt-get update

```
Now you need to install the python3-pip, use the following installation command to install the python3-pip (only need to run on ansible node .i.e. amaster)

```
sudo apt install python3-pip
```

After the installation verify the python and pip version
```
python -V
Python 2.7.15+
```
```
pip3 -V
pip 9.0.1 from /usr/lib/python3/dist-packages (python 3.6)
```

## Step 6: Clone the kubespray git repo (only need to run on ansible node .i.e. amaster)

```
git clone https://github.com/kubernetes-sigs/kubespray.git
```

## Step 7: Install kubespray package from “requirement.txt” (only need to run on ansible node .i.e. amaster)

Goto “kubespray” directory
```
cd kubespray
```
Install the kubespray packages
```
sudo pip3 install -r requirements.txt
```

## Step 8: Copy inventory file to current users (only need to run on ansible node .i.e. amaster)

```
cp -rfp inventory/sample inventory/mycluster
```

## Step 9: Prepare host.yml for kubespray (only need to run on ansible node .i.e. amaster)

This step is little trivial because we need to update host.yml with the nodes IP.

Now we are going to declare a variable “IPS” for storing the IP address of other nodes .i.e. kmaster(100.0.0.2), kworker(100.0.0.3)
```
declare -a IPS=(100.0.0.2 100.0.0.3)
```

```
CONFIG_FILE=inventory/mycluster/hosts.yml python3 contrib/inventory_builder/inventory.py ${IPS[@]}
```

After running the above commands do verify the hosts.yml and it should be like -
```
vi inventory/mycluster/hosts.yml
```

```
all:
  hosts:
    node1:
      ansible_host: 100.0.0.2
      ip: 100.0.0.2
      access_ip: 100.0.0.2
    node2:
      ansible_host: 100.0.0.3
      ip: 100.0.0.3
      access_ip: 100.0.0.3
  children:
    kube-master:
      hosts:
        node1:
        node2:
    kube-node:
      hosts:
        node1:
        node2:
    etcd:
      hosts:
        node1:
    k8s-cluster:
      children:
        kube-master:
        kube-node:
    calico-rr:
      hosts: {}

```

## Step 10: Run the ansible-playbook on ansible node .i.e. - amaster (only need to run on ansible node .i.e. amaster)

```
ansible-playbook -i inventory/mycluster/hosts.yml --become --become-user=root cluster.yml
```


## Step 11: Install kubectl on kubernetes master .i.e. - kmaster (only need to run on kuebernets node .i.e. kmaster)

Now you need to log into the kubernetes master .i.e. kmaster and download the kubectl onto it.
```
curl -LO https://storage.googleapis.com/kubernetes-release/release/`curl -s https://storage.googleapis.com/kubernetes-release/release/stable.txt`/bin/linux/amd64/kubectl
```

Now we need to copy the admin.conf file to .kube
```
sudo cp /etc/kubernetes/admin.conf /home/vagrant/config
```
```
mkdir .kube
```
```
mv config .kube/
```
```
sudo chown $(id -u):$(id -g ) $HOME/.kube/config
```

## Step 12: Verify the kubernetes nodes

```
kubectl get nodes
```
```
NAME    STATUS   ROLES    AGE   VERSION
node1   Ready    master   13m   v1.18.2
node2   Ready    master   13m   v1.18.2
```
