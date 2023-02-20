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
   
   for amd64
   
        curl -O https://s3.us-west-2.amazonaws.com/amazon-eks/1.24.10/2023-01-30/bin/linux/amd64/kubectl

    for arm64
    
        curl -O https://s3.us-west-2.amazonaws.com/amazon-eks/1.24.10/2023-01-30/bin/linux/arm64/kubectl
        
        
    Apply execute permissions to the binary.
    
        chmod +x ./kubectl

    Set the Path by copying to user Home Directory

        mkdir -p $HOME/bin && cp ./kubectl $HOME/bin/kubectl && export PATH=$PATH:$HOME/bin

    (Optional) Add the $HOME/bin path to your shell initialization file so that it is configured when you open a shell.
    
        echo 'export PATH=$PATH:$HOME/bin' >> ~/.bashrc
    
    After you install kubectl, you can verify its version.
    
        kubectl version --short --client

4. Install eksctl CLI

  a. Download and extract the latest release of eksctl with the following command.
      
        curl --silent --location "https://github.com/weaveworks/eksctl/releases/latest/download/eksctl_$(uname -s)_amd64.tar.gz" | tar xz -C /tmp

  b. Move the extracted binary to /usr/local/bin.
      
        sudo mv /tmp/eksctl /usr/local/bin
  
  c. Test that your installation was successful with the following command.
  
        eksctl version
        
        
#  Step 1 : Create EKS Cluster and Worker Nodes
  
- Create Cluster 
              
      eksctl create cluster --name=eksdemo1 \
                            --region=us-east-2 \
                            --zones=us-east-2a,us-east-2b \
                            --version="1.24" \
                            --without-nodegroup

- Get List of clusters 
  
      eksctl get cluster
      
      
- associate-iam-oidc-provider

Template 
        
    eksctl utils associate-iam-oidc-provider \
    --region region-code \
    --cluster <cluter-name> \
    --approve


- Replace with region & cluster name 

      eksctl utils associate-iam-oidc-provider \
      --region us-east-2 \
      --cluster eksdemo1 \
      --approve

- Create EKS NodeGroup in VPC Private Subnets 

       eksctl create nodegroup --cluster=eksdemo1 \
                        --region=us-east-2 \
                        --name=eksdemo1-ng-private1 \
                        --node-type=t2.medium \
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
                        --alb-ingress-access \
                        --node-private-networking

- Verify Cluster, Node Groups and configure kubectl cli if not configured
      1.	EKS Cluster
      2.	EKS Node Groups in Private Subnets


- Verfy EKS Cluster

      eksctl get cluster

- Verify EKS Node Groups

      eksctl get nodegroup --cluster=eksdemo1

- Configure kubeconfig for kubectl

      aws eks --region us-east-2 update-kubeconfig --name eksdemo1
      
- Verify EKS Nodes in EKS Cluster using kubectl

      kubectl get nodes
      
# Step 2 : Create IAM Policy

- Create IAM policy for the AWS Load Balancer Controller that allows it to make calls to AWS APIs on your behalf.

- As on today 2.4.5 is the latest Load Balancer Controller

- We will download always latest from main branch of Git Repo - https://github.com/kubernetes-sigs/aws-load-balancer-controller 


   a. Download IAM Policy

      curl -O https://raw.githubusercontent.com/kubernetes-sigs/aws-load-balancer-controller/v2.4.4/docs/install/iam_policy.json

  
   b. Create IAM Policy using policy downloaded 
   
   
      aws iam create-policy \
      --policy-name AWSLoadBalancerControllerIAMPolicy \
      --policy-document file://iam-policy.json

- Make a note of Policy ARN
  
  Make a note of Policy ARN as we are going to use that in next step when creating IAM Role


- Example : Policy ARN :  arn:aws:iam::865896445316:policy/AWSLoadBalancerControllerIAMPolicy


# Step 3 : Create an IAM role for the AWS LoadBalancer Controller and attach the role to the Kubernetes service account

   - Applicable only with eksctl managed clusters
   
   - This command will create an AWS IAM role
   
   - This command also will create Kubernetes Service Account in k8s cluster
   
   - In addition, this command will bound IAM Role created and the Kubernetes service account created

## Step 3A : Create IAM Role using eksctl

Now create using following commands :

#### Template

    eksctl create iamserviceaccount \
    --cluster=my_cluster \
    --namespace=kube-system \
    --name=aws-load-balancer-controller \                      #Note:  K8S Service Account Name that need to be bound to newly created IAM Role
    --attach-policy-arn=arn:aws:iam::111122223333:policy/AWSLoadBalancerControllerIAMPolicy \
    --override-existing-serviceaccounts \
    --approve

### Replaced name, cluster and policy arn (Policy arn we took note in step-2)

    eksctl create iamserviceaccount \
    --cluster=eksdemo1 \
    --namespace=kube-system \
    --name=aws-load-balancer-controller  \
    --attach-policy-arn=arn:aws:iam::865896445316:policy/AWSLoadBalancerControllerIAMPolicy\
    --override-existing-serviceaccounts \
    --approve	


## Step 3B : Verify using eksctl cli

Get IAM Service Account
    
     eksctl  get iamserviceaccount --cluster eksdemo1


## Step 3C : Verify k8s Service Account using kubectl
	
Verify if any existing service account

     kubectl get sa aws-load-balancer-controller  -n kube-system


Describe Service Account aws-load-balancer-controller

    kubectl describe sa aws-load-balancer-controller -n kube-system


# Step 4: Install the AWS Load Balancer Controller using Helm

## Step 4A : Install Helm
- Install Helm if not installed
- Install Helm for AWS EKS  Refereance : https://docs.aws.amazon.com/eks/latest/userguide/helm.html 

### 1. Run the appropriate command for your client operating system.

If you're using macOS with Homebrew, install the binaries with the following command.

    brew install helm

If you're using Windows with Chocolatey, install the binaries with the following command.

    choco install kubernetes-helm

If you're using Linux, install the binaries with the following commands.

    curl https://raw.githubusercontent.com/helm/helm/master/scripts/get-helm-3 > get_helm.sh
    chmod 700 get_helm.sh
    ./get_helm.sh

- Note : If you get a message that openssl must first be installed, you can install it with the following command.
    
      sudo yum install openssl

### 2. See the version of Helm that you installed.

        helm version --short | cut -d + -f 1

## Step 4B : Install AWS Load Balancer Controller

- Important-Note-1: If you're deploying the controller to Amazon EC2 nodes that have restricted access to the Amazon EC2 instance metadata service (IMDS), or if you're deploying to Fargate, then add the following flags to the command that you run:

    --set region=region-code
    --set vpcId=vpc-xxxxxxxx

- Important-Note-2: If you're deploying to any Region other than us-west-2, then add the following flag to the command that you run, replacing account and region-code with the values for your region listed in Amazon EKS add-on container image addresses.

Get Region Code and Account info:   https://docs.aws.amazon.com/eks/latest/userguide/add-ons-images.html 

--set image.repository=account.dkr.ecr.region-code.amazonaws.com/amazon/aws-load-balancer-controller

    
- Now to install controller  follow this commands :

a. Add the eks-charts repository.
 
    helm repo add eks https://aws.github.io/eks-charts

b. Update your local repo to make sure that you have the most recent charts.

    helm repo update
      
      
- Important : The deployed chart doesn't receive security updates automatically. You need to manually upgrade to a newer chart when it becomes available. When upgrading, change install to upgrade in the previous command, but run the following command to install the TargetGroupBinding custom resource definitions before running the previous command.

      kubectl apply -k "github.com/aws/eks-charts/stable/aws-load-balancer-controller/crds?ref=master"

c. Install the AWS Load Balancer Controller.

Template

    helm install aws-load-balancer-controller eks/aws-load-balancer-controller \
    -n kube-system \
    --set clusterName=<cluster-name> \
    --set serviceAccount.create=false \
    --set serviceAccount.name=aws-load-balancer-controller \
    --set region=<region-code> \
    --set vpcId=<vpc-xxxxxxxx> \			#get from VPC created for EKS
    --set image.repository=<account>.dkr.ecr.<region-code>.amazonaws.com/amazon/aws-load-balancer-controller 		## get this from repository as per your region

- Replace Cluster Name, Region Code, VPC ID, Image Repo Account ID and Region Code  
      
      helm install aws-load-balancer-controller eks/aws-load-balancer-controller \
      -n kube-system \
      --set clusterName=eksdemo1 \
      --set serviceAccount.create=false \
      --set serviceAccount.name=aws-load-balancer-controller \
      --set region=us-east-2 \
      --set vpcId=vpc-09a272ddfxxxxxxxxx \ 
      --set image.repository=602401143452.dkr.ecr.us-east-2.amazonws.com/amazon/aws-load-balancer-controller

## Step 4C: Verify that the controller is installed and Webhook Service created

  Verify that the controller is installed.
  
    kubectl -n kube-system get deployment 

  To Describe the Depoyment use following command 
  
    kubectl -n kube-system describe deployment aws-load-balancer-controller
   
    
  Verify AWS Load Balancer Controller Webhook service created

    kubectl -n kube-system get svc 
  
  To List out the pod use following command :
  
    kubectl -n kube-system get pods
  
  Step 4D : UNINSTALL AWS Load Balancer Controller using Helm Command (Information Purpose - SHOULD NOT EXECUTE THIS COMMAND)

This step should not be implemented.
This is just put it here for us to know how to uninstall aws load balancer controller from EKS Cluster

Uninstall AWS Load Balancer Controller

    helm uninstall aws-load-balancer-controller -n kube-system
