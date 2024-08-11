# Step by step on how to setup a cluster and connection via GUI
### via cli here: https://aws.plainenglish.io/a-guide-on-how-to-connect-to-your-aws-eks-cluster-f915466ac4e8

--- 

1. creating IAM rules to allow k8s(ref: https://docs.aws.amazon.com/eks/latest/userguide/service_IAM_role.html#create-service-role)
   - go to IAM
   - create a new service role specifying eks cluster (name will likely be: AmazonEKSClusterPolicy)
   - ![Screenshot_3](https://github.com/user-attachments/assets/7ce4342f-4904-4522-84ee-e0ffa68d122e)
   - give apropriate names in the necessary fields
   - and create the role
     
---

2. Creating the cluster
   - step 1(configure cluster):
     - choose a name for your cluster(mine is k8s-cluster1)
     - choose the cluster service role created in step one
     - ![Screenshot_1](https://github.com/user-attachments/assets/bda1aaf5-74a7-430d-9251-6de498f39ee9)
     - next
       
   - step 2(specify networking):
     - choose the VPC, Subnets and Security Group(I will go with the default)
     - ![Screenshot_4](https://github.com/user-attachments/assets/5150589b-b283-4dde-a296-c02b5eeb8fba)
     - next
       
   - step 3(configure observability):
     - ill leave it as default
     - next
       
   - step 4(select add-ons):
     - ![Screenshot_2](https://github.com/user-attachments/assets/29abe831-1a67-43d8-b3d3-793d02a8e527)
     - if not selected:
       - Amazon VPC CNI
       - Kube-Proxy
       - CoreDNS
       - Amazon EKS Pod Identity Agent
     - next  
    
    - step 5(configure selected add-ons settings):
      - select the versions you need, otherwise leave them as default
      - next
    
    - Create cluster and wait for it to be available(might take a while)

---

3. Configure the cluster
   - create a new IAM role for EC2 that will be used in the node-group(next step)
   - ![Screenshot_6](https://github.com/user-attachments/assets/614ec455-149d-434e-8df1-89a8bd369156)
     - attatch the following policies to the role:
       - AmazonEKSWorkerNodePolicy (Grants permissions required for worker nodes to connect to the EKS cluster)
       - AmazonEC2ContainerRegistryReadOnly (Allows nodes to pull container images from Amazon ECR)
       - AmazonEKS_CNI_Policy (Grants permissions to the Amazon VPC CNI plugin to manage networking)
       - AmazonS3ReadOnlyAccess (OPTIONALLY)
       - ![Screenshot_8](https://github.com/user-attachments/assets/88b8e67a-af43-46c9-a0a0-577331007b51)
     - assign a name and description
     - create the role
   - create a node-group and specify how many workers you want(this step is needed since the cluster relies on the node-group to know how many workers to create)
      - ![Screenshot_9](https://github.com/user-attachments/assets/d467d0b7-792a-4e46-9f52-3a6a24071775)
      - ![Screenshot_10](https://github.com/user-attachments/assets/1d3be86b-574e-45b4-9223-dd4c906de998)
      - ![Screenshot_11](https://github.com/user-attachments/assets/133f440e-3224-4e67-9c16-b01a5f645edc)

---

4. Manager creation and configuration
   - create a new EC2 instance in the same region as your cluster(I will create an ubuntu machine)
   - install AWS CLI in the instance(ref: https://docs.aws.amazon.com/cli/latest/userguide/getting-started-install.html)
   - to get the session token, use this site as reference: https://repost.aws/knowledge-center/authenticate-mfa-cli
     - aws sts get-session-token --serial-number arn-of-the-mfa-device --token-code code-from-token(mfa token)
     - MFA needs to be setup on the account to be able to use the command above
     - refer to this aws setup for the credentials and config files -> https://github.com/rlcosta177/personal-projects/blob/main/AWS/tf-aws-installation.md
   - install kubectl(done in ubuntu) ref: https://kubernetes.io/docs/tasks/tools/install-kubectl-linux/
   - configure the kubernetes cli to direct to your cluster
     - aws eks --region <your-region> update-kubeconfig --name my-eks-cluster
   - verify connection: kubectl get all
   - if you are still encountering errors:
     - try changing security group permissions
     - verify that your aws credentials file has up-to-date keys/token
   - sudo apt install build-essential
