#### MyProject #### <br>
----Install Terraform to my loacal machine <br>
wget https://releases.hashicorp.com/terraform/0.12.16/terraform_0.12.16_linux_amd64.zip <br>
unzip terraform_0.12.16_linux_amd64.zip <br>
sudo mv terraform /usr/local/bin/ && rm terraform_0.12.16_linux_amd64.zip <br>
terraform
--------Install and configure AWS CLI
yum install pip
pip install awscli

-------Configure the AWS CLI with the AWS credentials
--------AWS Access Key ID [None]: AKIAXYFTQRTPG6LKM27H 
--------AWS Secret Access Key [None]: 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 
-------Default region name [None]: ap-east-1
-------Default output format [None]: json
aws configure 
----------Install kubectl and wget 
curl -LO https://storage.googleapis.com/kubernetes-release/release/v1.18.0/bin/linux/amd64/kubectl
chmod +x ./kubectl
mv kubectl /usr/bin/
kubectl  version --client
yum install wget

----------Initialize terraform workspace 
git clone https://github.com/ashishrpandey/terraform-kubernetes-installation
cd terraform-kubernetes-installation

----------Initialize terraform workspace 
terraform init
terraform plan
terraform validate

----------Now i finished terraform installation and created the cluster
----------Cluster is active now at AWS

----------Install Jenkins, Ansible, and Docker
----------Install Python 3, Ansible, and the openshift module:
sudo apt update
sudo apt install -y python3
sudo apt install -y python3-pip
sudo pip3 install ansible
sudo pip3 install openshift
---------By default, pip installs binaries under a hidden directory in the user’s home folder. We need to add this directory to the $PATH variable so that we can easily call the command
echo "export PATH=$PATH:~/.local/bin" >> ~/.bashrc && . ~/.bashrc
---------Install the Ansible role necessary for deploying a Jenkins instance
---------Install docker role
ansible-galaxy install geerlingguy.jenkins
ansible-galaxy install geerlingguy.docker
--------Create the playbook.yaml

#########Deploying Jenkins on Amazon EKS with Amazon EFS#########
https://aws.amazon.com/blogs/storage/deploying-jenkins-on-amazon-eks-with-amazon-efs/

------Create an Amazon EFS file system and capture your VPC ID
aws ec2 describe-vpcs
-------.Create a security group for your Amazon EFS mount target
aws ec2 create-security-group 
--region us-east-1 
--group-name efs-mount-sg 
--description "Amazon EFS for EKS, SG for mount target" 
--vpc-id {Number}

-----------Add rules to the security group to authorize inbound/outbound access
aws ec2 authorize-security-group-ingress \
--group-id identifier for the security group created for our Amazon 
EFS mount targets (i.e. sg-1234567ab12a345cd) \
--region us-east-1 \
--protocol tcp \
--port 2049 \
--cidr 192.168.0.0/16

--------------Create an Amazon EFS file system
aws efs create-file-system \
--creation-token creation-token \
--performance-mode generalPurpose \
--throughput-mode bursting \
--region us-east-1 \
--tags Key=Name,Value=MyEFSFileSystem \
--encrypted

----------------Capture your VPC subnet IDs
aws ec2 describe-instances --filters Name=vpc-id,Values= identifier
for our VPC (i.e. vpc-1234567ab12a345cd) --query 
'Reservations[*].Instances[].SubnetId'

---------------Create two Amazon EFS mount targets
aws efs create-mount-target \
--file-system-id identifier for our file system (i.e. fs-123b45fa) \
--subnet-id identifier for our node group subnets (i.e. subnet-
1234567ab12a345cd) \
--security-group identifier for the security group created for our 
Amazon EFS mount targets (i.e. sg-1234567ab12a345cd) \
--region us-east-1

------------Create an Amazon EFS access point
aws efs create-access-point --file-system-id identifier for our file 
system (i.e. fs-123b45fa) \
--posix-user Uid=1000,Gid=1000 \
--root-directory 
"Path=/jenkins,CreationInfo={OwnerUid=1000,OwnerGid=1000,Permissions=777}"

---------------Deploy the Amazon EFS CSI driver to your Amazon EKS cluster
kubectl apply -k "github.com/kubernetes-sigs/aws-efs-csi-
driver/deploy/kubernetes/overlays/stable/?ref=master"

------------Output:
daemonset.apps/efs-csi-node created
csidriver.storage.k8s.io/efs.csi.aws.com created

------------Create efs-sc storage class YAML file
kind: StorageClass
apiVersion: storage.k8s.io/v1
metadata:
  name: efs-sc
provisioner: efs.csi.aws.com

-----------Create the efs-pv persistent volume YAML file
apiVersion: v1
kind: PersistentVolumeClaim
metadata:
  name: efs-claim
spec:
  accessModes:
    - ReadWriteMany
  storageClassName: efs-sc
  resources:
    requests:
      storage: 5Gi

----------Deploy the efs-sc storage class, efs-pv persistent volume, and efs-claim persistent volume claim
kubectl apply -f 
storageclass.yaml,persistentvolume.yaml,persistentvolumeclaim.yaml

----------Check to make sure the Kubernetes resources were created
kubectl get sc,pv,pvc

-------Deploy Jenkins to Amazon EKS
helm repo add stable https://kubernetes-charts.storage.googleapis.com/

------Install Jenkins on your EKS cluster
helm install jenkins stable/jenkins --set
rbac.create=true,master.servicePort=80,master.serviceType=LoadBalancer
,persistence.existingClaim=efs-claim

---------------Install sonarqube pods to the cluster--------------
helm install stable/sonarqube

https://www.jenkins.io/doc/book/installing/kubernetes/
