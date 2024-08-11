1. creating IAM rules to allow k8s(ref: https://docs.aws.amazon.com/eks/latest/userguide/service_IAM_role.html#create-service-role)
   - go to IAM
   - create a new service role specifying eks cluster (name will likely be: AmazonEKSClusterPolicy)
   - give apropriate names in the necessary fields
   - and create the role
 
2. Creating the cluster
   - step 1(configure cluster):
     - choose a name for your cluster(mine is k8s-cluster1)
     - choose the cluster service role created in step one
     - next
   - step 2(specify networking):
     - choose the VPC, Subnets and Security Group(I will go with the default)
     - next
   - step 3(configure observability):
     - ill leave it as default
     - next
   - step 4(select add-ons):
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

3. Configure the cluster
   - create a node-group and specify how many workers you want
