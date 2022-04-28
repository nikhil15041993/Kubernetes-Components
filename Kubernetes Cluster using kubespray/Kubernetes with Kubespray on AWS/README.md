

## Step 1.  Clone the kubespray git repo

```
git clone https://github.com/kubernetes-sigs/kubespray.git
```
Goto “kubespray” directory

## Step 2. Install the dependencies from requirements.text

```
sudo pip install -r requirements.txt
```

## Step 3. install Terraform 

download the latest version of Terraform according to your distribution and install it to your /usr/local/bin folder
```
wget https://releases.hashicorp.com/terraform/0.12.23/terraform_0.12.23_linux_amd64.zip
unzip terraform_0.12.23_linux_amd64.zip
sudo mv terraform /usr/local/bin/
```

## Step 4. Building a cloud infrastructure with Terraform

Enter the cloned directory and copy the credentials.
```
cd kubespray/contrib/terraform/aws/
cp credentials.tfvars.example credentials.tfvars
```





