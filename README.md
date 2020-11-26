# MyProject
#Install Terraform to my loacal machine
wget https://releases.hashicorp.com/terraform/0.12.16/terraform_0.12.16_linux_amd64.zip
unzip terraform_0.12.16_linux_amd64.zip
sudo mv terraform /usr/local/bin/ && rm terraform_0.12.16_linux_amd64.zip
terraform
#Install and configure AWS CLI
yum install pip
pip install awscli
#Configure the AWS CLI with the AWS credentials
aws configure 
#Install kubectl and wget 
curl -LO https://storage.googleapis.com/kubernetes-release/release/v1.18.0/bin/linux/amd64/kubectl
chmod +x ./kubectl
mv kubectl /usr/bin/
kubectl  version --client
yum install wget
#Initialize terraform workspace 
git clone https://github.com/ashishrpandey/terraform-kubernetes-installation
cd terraform-kubernetes-installation
#Initialize terraform workspace 
terraform init
terraform plan
terraform validate
#Now i finished terraform installation and created the cluster
#Cluster is active now at AWS
#Install Jenkins, Ansible, and Docker
#Install Python 3, Ansible, and the openshift module:
sudo apt update
sudo apt install -y python3
sudo apt install -y python3-pip
sudo pip3 install ansible
sudo pip3 install openshift
#By default, pip installs binaries under a hidden directory in the user’s home folder. We need to add this directory to the $PATH variable so that we can easily call the command
echo "export PATH=$PATH:~/.local/bin" >> ~/.bashrc && . ~/.bashrc
#Install the Ansible role necessary for deploying a Jenkins instance
#Install docker role
ansible-galaxy install geerlingguy.jenkins
ansible-galaxy install geerlingguy.docker
#Create the playbook.yaml
