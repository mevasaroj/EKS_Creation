# Private EKS Cluster Creation
1. Prerequisite
   - 
     - AWS Account
     - VPC & Subnet
     - AWS Command Line Interface (CLI)
     - GIT Source Code Management (SCM)
     - Terraform CLI / TFE Eenterprise
       
2. Network Prerequisite.
   -
   - It assume VPC and following Subnet are already Provisioned / If not Create the VPC and Subnet mentioned Below
     
     - Control Plane Subnet = Managed by AWS --> Primary CIDR RANGE
       - 
         - cp-subnet-aza - 10.x.x.x/28
         - cp-subnet-azb - 10.x.x.x/28
         - cp-subnet-azc - 10.x.x.x/28
      
     - Dataplane Subnet / Workernode Subnet = Managed by End User --> Primary CIDR RANGE
       - 
         - dp-subnet-aza - 10.x.x.x/24
         - dp-subnet-azb - 10.x.x.x/24
         - dp-subnet-azc - 10.x.x.x/24
           
     - Pods & Container Seondary Subnet = Managed by End User --> Secondary CIDR RANGE
       - 
         - pods-subnet-aza - 100.x.x.x/22
         - pods-subnet-azb - 100.x.x.x/22
         - pods-subnet-azc - 100.x.x.x/22
           
3. Create the following IAM Roles & Policy.
   - 
   - AWSServiceRoleForAmazonEKS Role for EKS Cluster
     - 
       - Open IAM --> Roles --> Create Role
       - Under = Select Trusted entity type
           - Trusted entity type : AWS service
           - Use cases : type "EKS" --> Select "EKS Service" --> First Option
       - Under Add permission : (Default) --> No Changes
       - Under Name, review, and create : Default = AWSServiceRoleForAmazonEKS --> No Changes
       - Create role
         
   - Cluster-Role for EKS Cluster Creation
     - 
       - Open IAM --> Roles --> Create Role
       - Under = Select Trusted entity type
           - Trusted entity type : AWS service
           - Use cases : type "EKS" --> Select "EKS Cluster" --> Second Option
       - Under Add permission : (Default) --> No Changes
       - Under Name, review, and create
           - Role Name : eks-cluster-role
           - Description : No Changes
           - Step 1: Select trusted entities : No Changes
           - Step 2: Add permissions: No Changes
           - Step 3: Add Tags : Add the require tags
       - Create Role

   - Add KMS Key Permission to Above cluster-role
     - 
     
   - WorkerNode-Role for Workernode (ec2)
     - 
       - Open IAM --> Roles --> Create Role
       - Under = Select Trusted entity type
           - Trusted entity type : AWS service
           - Use cases : type "ec2" --> Select "ec2" --> First Option
       - Under Add permission : Add Following Permission policies
           - AmazonEKSWorkerNodePolicy
           - AmazonEC2ContainerRegistryReadOnly
           - AmazonSSMManagedInstanceCore
           - AmazonEKS_CNI_Policy
           - AmazonEFSCSIDriverPolicy --> Optional for EFS if same role using for EFS for Pods as external Device
           - AmazonEBSCSIDriverPolicy --> Optional for EBS if same role using for EBS for Pods as external Device
           - AmazonEC2RoleforSSM --> Optional for SSM
       - Under Name, review, and create
           - Role Name : eks-workernode-role
           - Description : No Changes
           - Step 1: Select trusted entities : No Changes
           - Step 2: Add permissions: No Changes
           - Step 3: Add Tags : Add the require tags
       - Create Role.
   - Add KMS Key Permission to Above WorkerNode Role
     - 
   - [Edit the WorkerNode-Role Trust-Relationship as Mention Below](https://github.com/mevasaroj/EKS_Creation/blob/main/WorkerNode-Role-trust-Relationship.txt)
     
  
4. Create the following Security Group for Private connection.
   - 
   - eks-cluster-addition-security-group
     -
     -
     -
     -
     - a
   - eks-cluster-workernode-security-group
     -
     -
     -
     -
     - a
   - vpc-endpoint-security-group
     -
     -
     -
     - a





5. Create the following VPC endpoints for Private connection.
   -
   - com.amazonaws.region.s3 - (Gateway Type)
     -
     -
     -
     -
     - a
   - com.amazonaws.region.ec2 (Interface)
     -
     -
     -
     -
     - a
   - Create following interface endpoint as created above
     - 
     - com.amazonaws.region.ecr.api (Interface)
     - com.amazonaws.region.ecr.dkr (Interface)
     - com.amazonaws.region.sts (Interface)
     - com.amazonaws.region-code.eks (Interface)
     - com.amazonaws.region-code.eks-auth (Interface)
     -
     -
     -
     - a

  
9. 
  

   -
   -
   -
   -
   -
   - a

2. Network Prerequisite


4.
8.
9.
10.
11.
12.
13.
14.
13.
14.
15.
16.
17.
18.
19.
20.
21.
22. a 
2. 
  


  2.2. 




  
   - Add KMS Key Policy to  eks-workernode-role
    - 
   - Edit Trust RelationShip for eks-workernode-role as follow




    -
    -
  C. Following Security and Rule Must be created.


  D. Following VPC Endpoint Must be created.


      
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
  
