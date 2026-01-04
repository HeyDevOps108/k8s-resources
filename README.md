--------------------------------------------------
IAM ROLES & PERMISSIONS (IMPORTANT)
--------------------------------------------------

AWS EKS requires multiple IAM roles.
These roles are created either automatically or manually
during cluster and nodegroup setup.

--------------------------------------------------
1. EKS CLUSTER SERVICE ROLE
--------------------------------------------------

Purpose:
- Allows EKS control plane to call AWS services
- Required to create and manage the cluster

AWS Console Path:
IAM → Roles → Create role

Selections:
- Trusted entity: AWS service
- Use case: EKS
- Role type: EKS - Cluster

Permissions (auto-attached):
- AmazonEKSClusterPolicy

Role Name (example):
- eks-cluster-role-cndevcorp5

This role is selected during:
EKS → Create Cluster → Cluster service role

--------------------------------------------------
2. NODEGROUP IAM ROLE (WORKER NODES ROLE)
--------------------------------------------------

Purpose:
- Allows EC2 worker nodes to:
  - Join the EKS cluster
  - Pull images from ECR
  - Communicate with AWS services

AWS Console Path:
IAM → Roles → Create role

Selections:
- Trusted entity: AWS service
- Use case: EC2

Permissions to attach:
- AmazonEKSWorkerNodePolicy
- AmazonEKS_CNI_Policy
- AmazonEC2ContainerRegistryReadOnly

Role Name (example):
- eks-nodegroup-role-cndevcorp5

This role is selected during:
EKS → Cluster → Compute → Add Node Group

--------------------------------------------------
3. JENKINS / MASTER SERVER IAM ROLE
--------------------------------------------------

Purpose:
- Allows Jenkins to deploy applications to EKS
- Allows Jenkins to push/pull images from ECR

Recommended Approach:
- Attach IAM role to EC2 instance
- Avoid hardcoding AWS keys

AWS Console Path:
EC2 → Instances → Select master-tools-server
→ Actions → Security → Modify IAM role

Permissions to attach:
- AmazonEKSClusterPolicy
- AmazonEKSWorkerNodePolicy
- AmazonEC2ContainerRegistryPowerUser
- IAMReadOnlyAccess (optional, for visibility)

This allows Jenkins to:
- Run kubectl
- Authenticate to EKS
- Push images to ECR

--------------------------------------------------
4. KUBECONFIG & IAM AUTHENTICATION
--------------------------------------------------

EKS authentication uses IAM.

Command used:
aws eks update-kubeconfig --region us-east-1 --name cndevcorp5

Important:
- Whenever EKS cluster is recreated,
  kubeconfig must be regenerated
- Jenkins user must have the updated kubeconfig

Copy kubeconfig to Jenkins:
sudo mkdir -p /var/lib/jenkins/.kube
sudo cp ~/.kube/config /var/lib/jenkins/.kube/config
sudo chown -R jenkins:jenkins /var/lib/jenkins/.kube

--------------------------------------------------
5. IMPORTANT NOTES
--------------------------------------------------

- kubectl does NOT manage IAM
- eksctl / AWS Console manages IAM roles
- Never delete IAM roles while cluster is running
- If nodegroup fails to join cluster,
  IAM role permissions should be checked first