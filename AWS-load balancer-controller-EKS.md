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
      
# Step-02 : Create IAM Policy











