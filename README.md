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
     
 - EKS Cluster provisionning
  - Define all variables for your EKS CloudFormation stack :
     - export EKS_STACK_NAME="eks"
     - export EKS_AWS_REGION="eu-west-3"
     - export EKS_KEY_PAIR_NAME="my-eks-key"
  - Apply the YAML file to provision
