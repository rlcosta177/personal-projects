# Step-by-Step Guide: Setting Up an AWS EKS Cluster and Connecting via GUI

For CLI instructions, refer to the following guide: [A Guide on How to Connect to Your AWS EKS Cluster](https://aws.plainenglish.io/a-guide-on-how-to-connect-to-your-aws-eks-cluster-f915466ac4e8).

---

### 1. Creating IAM Roles for Kubernetes
Reference: [AWS EKS IAM Role Guide](https://docs.aws.amazon.com/eks/latest/userguide/service_IAM_role.html#create-service-role)
   - Navigate to **IAM** in the AWS console.
   - Create a new **service role** for EKS (likely named `AmazonEKSClusterPolicy`).
   - ![IAM Role Screenshot](https://github.com/user-attachments/assets/7ce4342f-4904-4522-84ee-e0ffa68d122e)
   - Fill in the necessary fields with appropriate names.
   - Complete the role creation process.

---

### 2. Creating the EKS Cluster
   - **Step 1: Configure Cluster**:
     - Choose a name for your cluster (e.g., `k8s-cluster1`).
     - Select the IAM service role created in step 1.
     - ![Cluster Config Screenshot](https://github.com/user-attachments/assets/bda1aaf5-74a7-430d-9251-6de498f39ee9)
     - Click **Next**.

   - **Step 2: Specify Networking**:
     - Choose the VPC, subnets, and security group (using the default is fine).
     - ![Networking Config Screenshot](https://github.com/user-attachments/assets/5150589b-b283-4dde-a296-c02b5eeb8fba)
     - Click **Next**.

   - **Step 3: Configure Observability**:
     - Leave settings as default (unless needed otherwise).
     - Click **Next**.

   - **Step 4: Select Add-ons**:
     - ![Add-ons Screenshot](https://github.com/user-attachments/assets/29abe831-1a67-43d8-b3d3-793d02a8e527)
     - Ensure the following are selected:
       - Amazon VPC CNI
       - Kube-Proxy
       - CoreDNS
       - Amazon EKS Pod Identity Agent
     - Click **Next**.

   - **Step 5: Configure Add-on Settings**:
     - Select the versions needed, or leave them as default.
     - Click **Next**.

   - Click **Create Cluster** and wait for the cluster to be available (this may take some time).

---

### 3. Configuring the Cluster
   - Create a new IAM role for EC2 (this will be used in the node group created in the next step).
   - ![IAM Role Screenshot](https://github.com/user-attachments/assets/614ec455-149d-434e-8df1-89a8bd369156)
     - Attach the following policies to the role:
       - `AmazonEKSWorkerNodePolicy` (Allows worker nodes to connect to the EKS cluster)
       - `AmazonEC2ContainerRegistryReadOnly` (Allows nodes to pull container images from Amazon ECR)
       - `AmazonEKS_CNI_Policy` (Grants permissions to the Amazon VPC CNI plugin)
       - `AmazonS3ReadOnlyAccess` (Optional)
       - ![Policies Screenshot](https://github.com/user-attachments/assets/88b8e67a-af43-46c9-a0a0-577331007b51)
     - Assign a name and description to the role, then create it.

   - Create a **node group** and specify the number of worker nodes required.
     - ![Node Group Screenshot 1](https://github.com/user-attachments/assets/d467d0b7-792a-4e46-9f52-3a6a24071775)
     - ![Node Group Screenshot 2](https://github.com/user-attachments/assets/1d3be86b-574e-45b4-9223-dd4c906de998)
     - ![Node Group Screenshot 3](https://github.com/user-attachments/assets/133f440e-3224-4e67-9c16-b01a5f645edc)

---

### 4. Manager Creation and Configuration
   - Create a new **EC2 instance** in the same region as your cluster (Ubuntu is recommended).
   - Install AWS CLI on the instance. Reference: [AWS CLI Installation Guide](https://docs.aws.amazon.com/cli/latest/userguide/getting-started-install.html).
   - Obtain the session token by referring to this [AWS guide](https://repost.aws/knowledge-center/authenticate-mfa-cli):
     ```bash
     aws sts get-session-token --serial-number arn-of-the-mfa-device --token-code code-from-token
     ```
     - Note: MFA must be set up on the account to run the command above.
     - Refer to this [AWS credentials setup guide](https://github.com/rlcosta177/personal-projects/blob/main/AWS/tf-aws-installation.md) for configuring credentials and the config file.

   - Install **kubectl** on your EC2 instance (Ubuntu). Reference: [Install kubectl on Linux](https://kubernetes.io/docs/tasks/tools/install-kubectl-linux/).
   
   - Configure `kubectl` to connect to your cluster:
     ```bash
     aws eks --region <your-region> update-kubeconfig --name my-eks-cluster
     ```

   - Verify the connection:
     ```bash
     kubectl get all
     ```

   - If you encounter errors:
     - Check the security group permissions.
     - Ensure your AWS credentials file contains the correct keys and token.
   
   - Install essential build tools (if needed):
     ```bash
     sudo apt install build-essential
     ```
