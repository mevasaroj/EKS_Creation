# EKS_Creation
1 - Prerequisite
 - AWS Account
 - AWS Command Line Interface (CLI)
 - GIT Source Code Management (SCM)
 - Kubectl
 
2 - Infrastructure Deployment
 - Networking Setup
   - VPC Creation : Create a VPC called eks-VPC 
   - Internet Gateway: Internet Gateway (IGW) is a resource that allows communication between your VPC and internet.
   - Route Table: A public route table is necessary to declare all routes that will be used by the VPC. 
   - Subnets creation: Created 4 Subnets - 2 Public and 2 Private
   - To allow internet access for worker nodes from each subnet it's necessary to associate each Public Subnet to the eks-RouteTable. 
   - Security Groups: SG is a set of rules with fine granularity to allow communication towards a resource
 
 - AWS EKS Cluster
   - EKS IAM Cluster Role: To Provide the access to the other resources  in AWS we must need to create EKS IAM Cluster Role.
   - EKS IAM Node Group Role: The Amazon EKS node kubelet daemon need IAM instance profile to communication 
 
3 - Infrastructure Provision 
 - Clone the Git Repository
    - git clone https://github.com/mevasaroj/EKS_Creation.git $HOME/aws-eks
    - cd $HOME/aws-eks/
 
  - Define all variables for your EKS CloudFormation stack :
     - export EKS_STACK_NAME="eks"
     - export EKS_AWS_REGION="eu-west-3"
     - export EKS_KEY_PAIR_NAME="my-eks-key"
    
  - Create an EKS Key Pair :
     - aws ec2 create-key-pair \
  --region $EKS_AWS_REGION \
  --key-name $EKS_KEY_PAIR_NAME \
  --tag-specifications 'ResourceType=key-pair,Tags=[{Key=Name,Value=eks-key-pair},{Key=Project,Value=aws-eks}]' \
  --output text \
  --query 'KeyMaterial' > $HOME/aws-eks/infrastructure-as-code/eks.id_rsa

  - Create the EKS Cluster
    - aws cloudformation create-stack --stack-name $EKS_STACK_NAME \
  --region $EKS_AWS_REGION \
  --template-body file://$HOME/aws-eks/eks-cloudformation.yaml  \
  --capabilities CAPABILITY_NAMED_IAM
  
 - Please note that the EKS Cluster can take up to 15-20 minutes to complete.
 - When the loop is terminated, ensure the EKS Cluster status is ACTIVE :
   - aws eks --region $EKS_AWS_REGION describe-cluster \
  --name $EKS_CLUSTER_NAME \
  --query "cluster.status" \
  --output text
  
 4 - EKS Cluster testing
  - first step is to generate and retrieve the kubernetes configuration file with credentials 
    - aws eks \
  --region $EKS_AWS_REGION update-kubeconfig \
  --name $EKS_CLUSTER_NAME
  
  - operate the EKS Cluster
    - kubectl get node
  
  - Create an app-shark pod :
    - kubectl run app-shark \
  --image=sokubedocker/shark-application:eks \
  --restart=Never 
  
  - Create a LoadBalancer service on port 8080
    - kubectl expose pod app-shark --type=LoadBalancer --port=8080
  - The ELB creation takes usually nearly 5 minutes before it can be used.

  - Once the ELB is created. We will validate
    - curl http://$EKS_ELB_HOSTNAME:8080
  
