# Install AWS, kubectl & eksctl CLI's


1. Install AWS latest CLI
  
        curl "https://awscli.amazonaws.com/awscli-exe-linux-x86_64.zip" -o "awscliv2.zip"
        unzip awscliv2.zip
        sudo ./aws/install

2. Configure AWS Command Line using Security Credentials
    Go to AWS Management Console --> Services --> IAM
    
    Select the IAM User: eksctl-user
    
    Important Note: Use only IAM user to generate Security Credentials. Never ever use Root User. (Highly not recommended)
    
    Click on Security credentials tab
    
    Click on Create access key
    
    Copy Access ID and Secret access key
    
    Go to command line and provide the required details

3. Install kubectl CLI

   Download the kubectl binary for your cluster's Kubernetes version from Amazon S3 using the command for your device's hardware platform. The first link for each        version is for amd64 and the second link is for arm64.
   
A. Download Binaries as per operatinf system
   
   for amd64
   
        curl -O https://s3.us-west-2.amazonaws.com/amazon-eks/1.24.10/2023-01-30/bin/linux/amd64/kubectl

   for arm64
    
         curl -O https://s3.us-west-2.amazonaws.com/amazon-eks/1.24.10/2023-01-30/bin/linux/arm64/kubectl
     
     
        
B.  Apply execute permissions to the binary.
    
        chmod +x ./kubectl

C.  Set the Path by copying to user Home Directory

        mkdir -p $HOME/bin && cp ./kubectl $HOME/bin/kubectl && export PATH=$PATH:$HOME/bin

D.  (Optional) Add the $HOME/bin path to your shell initialization file so that it is configured when you open a shell.
    
        echo 'export PATH=$PATH:$HOME/bin' >> ~/.bashrc
    
E.  After you install kubectl, you can verify its version.
    
        kubectl version --short --client

4. Install eksctl CLI

  a. Download and extract the latest release of eksctl with the following command.
      
        curl --silent --location "https://github.com/weaveworks/eksctl/releases/latest/download/eksctl_$(uname -s)_amd64.tar.gz" | tar xz -C /tmp

  b. Move the extracted binary to /usr/local/bin.
      
        sudo mv /tmp/eksctl /usr/local/bin
  
  c. Test that your installation was successful with the following command.
  
        eksctl version


# Create EKS Cluster & Node Groups

- STEP 1 : Create EKS Cluster using eksctl, It will take 15 to 20 minutes to create the Cluster Control Plane.

        eksctl create cluster --name=eksdemo1 \
                        --region=us-east-2 \
                        --zones=us-east-2a,us-east-2b \
                        --without-nodegroup

Get List of clusters
            
    eksctl get cluster

- STEP 2 : Create & Associate IAM OIDC Provider for our EKS Cluster

     •	To enable and use AWS IAM roles for Kubernetes service accounts on our EKS cluster, we must create & associate OIDC identity provider.

     •	To do so using eksctl we can use the below command.

     •	Use latest eksctl version (as on today the latest version is 0.124.0)


      eksctl utils associate-iam-oidc-provider \
      --region us-east-2 \
      --cluster eksdemo1 \
      --approve

- STEP 3 : Create EC2 Keypair 
     
     •	Create a new EC2 Keypair with name as kube-demo
     
     •	This keypair we will use it when creating the EKS NodeGroup.
     
     •	This will help us to login to the EKS Worker Nodes using Terminal

- STEP 4 : Create Node Group with additional Add-Ons in Public Subnets

     •	These add-ons will create the respective IAM policies for us automatically within our Node Group role.
     
     
 Create Public Node Group 
 
        eksctl create nodegroup --cluster=eksdemo1 \
                       --region=us-east-2 \
                       --name=eksdemo1-ng-public1 \
                       --node-type=t2.medium \
                       --nodes=2 \
                       --nodes-min=2 \
                       --nodes-max=4 \
                       --node-volume-size=20 \
                       --ssh-access \
                       --ssh-public-key=kube-demo \
                       --managed \
                       --asg-access \
                       --external-dns-access \
                       --full-ecr-access \
                       --appmesh-access \
                       --alb-ingress-access

- STEP 5 : Verify Cluster & Nodes

    •	Verify NodeGroup subnets to confirm EC2 Instances are in Public Subnet

    •	Verify the node group subnet to ensure it created in public subnets


    Go to Services -> EKS -> eksdemo -> eksdemo1-ng1-public

    Click on Associated subnet in Details tab

    Click on Route Table Tab.

    We should see that internet route via Internet Gateway (0.0.0.0/0 -> igw-xxxxxxxx)

    Verify Cluster, NodeGroup in EKS Management Console

    Go to Services -> Elastic Kubernetes Service -> eksdemo1


List EKS clusters
        
          eksctl get cluster

List NodeGroups in a cluster
     
     eksctl get nodegroup --cluster=eksdemo1

List Nodes in current kubernetes cluster

     kubectl get nodes -o wide
     
    
- NOTE : To use kubectl command you have to update kubeconfig with  cluster and region in which cluster is active.


      aws eks --region us-east-2 update-kubeconfig --name eksdemo1
