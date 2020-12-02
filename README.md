<h1> MyProject </h1> <br>

<strong> Subject :  DevOps Assignment </strong> <br>
<strong> Author  :  Heba Mohamed </strong>
<img src="https://miro.medium.com/max/1076/1*QwJOyLmOeKOSCmCNXw1CUg.png" width=200px heigth=200px />
<h3> <strong> Install Terraform to my loacal machine </strong> <h3> <h3> 

> $ wget https://releases.hashicorp.com/terraform/0.12.16/terraform_0.12.16_linux_amd64.zip <br>
> $ unzip terraform_0.12.16_linux_amd64.zip <br>
> $ sudo mv terraform /usr/local/bin/ && rm terraform_0.12.16_linux_amd64.zip <br>
> $ terraform
<h3> Install and configure AWS CLI </h3> <br>
> $ yum install pip
> $ pip install awscli

<h3> Configure the AWS CLI with the AWS credentials </h3> <br>
<h3> AWS Access Key ID [None]: AKIAXYFTQRTPG6LKM27H  </h3> <br>
<h3> AWS Secret Access Key [None]: 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  </h3> <br>
<h3> Default region name [None]: ap-east-1 </h3> <br>
<h3> Default output format [None]: json </h3> <br>
> $ aws configure <br>
<h3> Install kubectl and wget <br>
> $ curl -LO https://storage.googleapis.com/kubernetes-release/release/v1.18.0/bin/linux/amd64/kubectl<br>
> $ chmod +x ./kubectl <br>
> $ mv kubectl /usr/bin/ <br>
> $ kubectl  version --client <br>
> $ yum install wget <br>

<h3> Initialize terraform workspace  </h3> <br>
> $ git clone https://github.com/ashishrpandey/terraform-kubernetes-installation<br>
> $ cd terraform-kubernetes-installation <br>

<h3> Initialize terraform workspace  </h3> <br>
> $ terraform init <br>
> $ terraform plan <br>
> $ terraform validate <br>

<h3> Now i finished terraform installation and created the cluster </h3> <br>
<h3> Cluster is active now at AWS </h3> <br>

<h3> Install Jenkins, Ansible, and Docker </h3> <br>
<h3> Install Python 3, Ansible, and the openshift module: </h3> <br>
> $ sudo apt update <br>
> $ sudo apt install -y python3 <br>
> $ sudo apt install -y python3-pip <br>
> $ sudo pip3 install ansible <br>
> $ sudo pip3 install openshift <br>
<h3> By default, pip installs binaries under a hidden directory in the user’s home folder. We need to add this directory to the $PATH variable so that we can easily call the command </h3> <br>
> $ echo "export PATH=$PATH:~/.local/bin" >> ~/.bashrc && . ~/.bashrc <br>
<h3> Install the Ansible role necessary for deploying a Jenkins instance </h3> <br>
<h3> Install docker role </h3> <br>
> $ ansible-galaxy install geerlingguy.jenkins <br>
> $ ansible-galaxy install geerlingguy.docker <br>
<h3> Create the playbook.yaml </h3> <br>

#########Deploying Jenkins on Amazon EKS with Amazon EFS######### <br>
> $ https://aws.amazon.com/blogs/storage/deploying-jenkins-on-amazon-eks-with-amazon-efs/ <br>

<h3> Create an Amazon EFS file system and capture your VPC ID </h3> <br>
> $ aws ec2 describe-vpcs <br>
<h3> .Create a security group for your Amazon EFS mount target </h3> <br>
> $ aws ec2 create-security-group  <br>
<h3> region us-east-1  <br>
<h3> group-name efs-mount-sg  <br>
<h3> description "Amazon EFS for EKS, SG for mount target" <br>
<h3> vpc-id {Number} <br>

<h3> Add rules to the security group to authorize inbound/outbound access </h3> <br>
> $ aws ec2 authorize-security-group-ingress \  <br>
> group-id identifier for the security group created for our Amazon <br>
> EFS mount targets (i.e. sg-1234567ab12a345cd) \ <br>
> --region us-east-1 \  <br>
> --protocol tcp \  <br>
> --port 2049 \ <br>
> --cidr 192.168.0.0/16 <br>

<h3> Create an Amazon EFS file system </h3> <br>
> $ aws efs create-file-system \<br>
> --creation-token creation-token \<br>
> --performance-mode generalPurpose \<br>
> --throughput-mode bursting \<br>
> --region us-east-1 \<br>
> --tags Key=Name,Value=MyEFSFileSystem \<br>
> --encrypted<br>

<h3> Capture your VPC subnet IDs </h3> <br>
> $ aws ec2 describe-instances --filters Name=vpc-id,Values= identifier <br>
> for our VPC (i.e. vpc-1234567ab12a345cd) --query 
> 'Reservations[*].Instances[].SubnetId' <br>

<h3> Create two Amazon EFS mount targets </h3> <br>
> $ aws efs create-mount-target \ <br>
>--file-system-id identifier for our file system (i.e. fs-123b45fa) \ <br>
> --subnet-id identifier for our node group subnets (i.e. subnet- <br>
> 1234567ab12a345cd) \ <br>
> --security-group identifier for the security group created for our  <br>
> Amazon EFS mount targets (i.e. sg-1234567ab12a345cd) \ <br>
> --region us-east-1 <br>

<h3> Create an Amazon EFS access point </h3> <br>
> $ aws efs create-access-point --file-system-id identifier for our file  <br>
> system (i.e. fs-123b45fa) \ <br>
> --posix-user Uid=1000,Gid=1000 \ <br>
> --root-directory  <br>
> "Path=/jenkins,CreationInfo={OwnerUid=1000,OwnerGid=1000,Permissions=777}" <br>

<h3> Deploy the Amazon EFS CSI driver to your Amazon EKS cluster </h3> <br>
> $ kubectl apply -k "github.com/kubernetes-sigs/aws-efs-csi- <br>
> driver/deploy/kubernetes/overlays/stable/?ref=master" <br>

<h3> Output: </h3> <br>
daemonset.apps/efs-csi-node created <br>
csidriver.storage.k8s.io/efs.csi.aws.com created <br>

<h3> Create efs-sc storage class YAML file </h3> <br>
> kind: StorageClass <br>
> apiVersion: storage.k8s.io/v1 <br>
> metadata: <br>
> name: efs-sc <br>
> provisioner: efs.csi.aws.com <br>

<h3> Create the efs-pv persistent volume YAML file </h3> <br>
> apiVersion: v1 <br>
> kind: PersistentVolumeClaim <br>
> metadata: <br>
>  name: efs-claim <br>
> spec: <br>
  accessModes: <br>
    - ReadWriteMany <br>
  storageClassName: efs-sc <br>
  resources: <br>
    requests: <br>
      storage: 5Gi <br>

<h3> Deploy the efs-sc storage class, efs-pv persistent volume, and efs-claim persistent volume claim </h3> <br>
> $ kubectl apply -f  <br>
> storageclass.yaml,persistentvolume.yaml,persistentvolumeclaim.yaml <br>

<h3> Check to make sure the Kubernetes resources were created </h3> <br>
> $ kubectl get sc,pv,pvc <br>

<h3> Deploy Jenkins to Amazon EKS </h3> <br>
> $ helm repo add stable https://kubernetes-charts.storage.googleapis.com/ <br>

<h3> Install Jenkins on your EKS cluster </h3> <br>
> $ helm install jenkins stable/jenkins --set <br>
rbac.create=true,master.servicePort=80,master.serviceType=LoadBalancer <br>
,persistence.existingClaim=efs-claim <br>

<h3> Install sonarqube pods to the cluster </h3> <br>
> $ helm install stable/sonarqube <br>

https://www.jenkins.io/doc/book/installing/kubernetes/ <br>
