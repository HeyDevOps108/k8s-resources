AWS EKS COMPLETE SETUP – START TO END (PLAIN TEXT)

============================================================
OVERVIEW
============================================================

This document contains end-to-end steps to set up AWS EKS
for learning / non-production use, including:

- Tool installation
- IAM roles
- EKS cluster creation
- kubeconfig setup
- Nodegroup creation
- Namespace creation
- Jenkins integration
- Common issues and fixes

Cluster Name used: cndevcorp5
Region: us-east-1

============================================================
PREREQUISITES
============================================================

1. AWS account with required permissions
2. EC2 instance (master / Jenkins server)
3. Internet access from EC2
4. IAM role attached to EC2 OR AWS access keys

============================================================
INSTALL AWS CLI (v2)
============================================================

curl "https://awscli.amazonaws.com/awscli-exe-linux-x86_64.zip" -o "awscliv2.zip"
sudo apt update
sudo apt install -y unzip
unzip awscliv2.zip
sudo ./aws/install

Verify:
aws --version

============================================================
CONFIGURE AWS ACCESS
============================================================

aws configure

- AWS Access Key ID
- AWS Secret Access Key
- Default region: us-east-1
- Output format: json

Verify:
aws sts get-caller-identity

============================================================
INSTALL kubectl
============================================================

curl -LO https://storage.googleapis.com/kubernetes-release/release/`curl -s https://storage.googleapis.com/kubernetes-release/release/stable.txt`/bin/linux/amd64/kubectl
chmod +x kubectl
sudo mv kubectl /usr/local/bin/

Verify:
kubectl version --client

============================================================
INSTALL eksctl
============================================================

curl --silent --location "https://github.com/weaveworks/eksctl/releases/latest/download/eksctl_$(uname -s)_amd64.tar.gz" | tar xz -C /tmp
sudo mv /tmp/eksctl /usr/local/bin

Verify:
eksctl version

============================================================
IAM ROLES (IMPORTANT)
============================================================

------------------------------------------------------------
1. EKS CLUSTER SERVICE ROLE
------------------------------------------------------------

AWS Console:
IAM → Roles → Create role

Trusted entity:
- AWS service
- Use case: EKS
- Role type: EKS - Cluster

Permissions (auto attached):
- AmazonEKSClusterPolicy

Example Role Name:
- eks-cluster-role-cndevcorp5

This role is selected during cluster creation.

------------------------------------------------------------
2. NODEGROUP IAM ROLE (WORKER NODES)
------------------------------------------------------------

AWS Console:
IAM → Roles → Create role

Trusted entity:
- AWS service
- Use case: EC2

Attach these policies:
- AmazonEKSWorkerNodePolicy
- AmazonEKS_CNI_Policy
- AmazonEC2ContainerRegistryReadOnly

Example Role Name:
- eks-nodegroup-role-cndevcorp5

This role is selected during nodegroup creation.

------------------------------------------------------------
3. JENKINS / MASTER SERVER IAM ROLE
------------------------------------------------------------

Recommended:
Attach IAM role directly to EC2 instance.

AWS Console:
EC2 → Instances → Select instance
→ Actions → Security → Modify IAM role

Attach policies:
- AmazonEKSClusterPolicy
- AmazonEC2ContainerRegistryPowerUser
- AmazonEKSWorkerNodePolicy

This allows Jenkins to:
- Access EKS
- Run kubectl
- Push/pull images from ECR

============================================================
CREATE EKS CLUSTER (CONTROL PLANE)
============================================================

AWS Console:
EKS → Create cluster

Settings:
- Cluster name: cndevcorp5
- Kubernetes version: 1.29
- Cluster service role: select EKS cluster role
- Endpoint access: Public + Private
- Select ALL public and private subnets
- Observability: Disabled
- Add-ons: Default

Wait until cluster status becomes ACTIVE.

============================================================
CONFIGURE kubeconfig
============================================================

aws eks update-kubeconfig --region us-east-1 --name cndevcorp5

Verify:
kubectl cluster-info
kubectl get nodes

(Expected: No resources found)

============================================================
FIX kubectl TIMEOUT ISSUE (IF ANY)
============================================================

If kubectl times out to 10.x.x.x:

1. Go to:
   EKS → Cluster → Networking
2. Find Cluster Security Group
3. Add inbound rule:
   - HTTPS (443)
   - Source: Security Group of master server

Also ensure VPC settings:
- DNS resolution: Enabled
- DNS hostnames: Enabled

============================================================
CREATE NODEGROUP (SIMPLE COMMAND)
============================================================

eksctl create nodegroup \
  --cluster cndevcorp5 \
  --region us-east-1 \
  --name fsgbu-obcbcs-preprod \
  --node-type t3.medium \
  --nodes 2 \
  --nodes-min 1 \
  --nodes-max 2 \
  --managed

Wait until nodegroup creation completes.

Verify:
kubectl get nodes

============================================================
OPTIONAL: LABEL NODES (LOGICAL NODEGROUP)
============================================================

kubectl label node <node-name> nodegroup=fsgbu-obcbcs-preprod env=preprod

Verify:
kubectl get nodes --show-labels

============================================================
CREATE NAMESPACE (SUBNAMESPACE STYLE)
============================================================

apiVersion: v1
kind: Namespace
metadata:
  name: fsgbu-obcbcs-preprod--foundation
  labels:
    env: preprod
    nodegroup: fsgbu-obcbcs-preprod

Apply:
kubectl apply -f namespace.yaml

============================================================
JENKINS kubeconfig FIX (VERY IMPORTANT)
============================================================

Whenever cluster is recreated, EKS endpoint changes.

Copy fresh kubeconfig to Jenkins user:

sudo mkdir -p /var/lib/jenkins/.kube
sudo cp ~/.kube/config /var/lib/jenkins/.kube/config
sudo chown -R jenkins:jenkins /var/lib/jenkins/.kube

Verify:
sudo -u jenkins kubectl cluster-info

============================================================
COMMON ISSUES & FIXES
============================================================

Issue:
kubectl openapi / DNS error

Reason:
kubeconfig pointing to old EKS endpoint

Fix:
aws eks update-kubeconfig
copy kubeconfig to Jenkins user

------------------------------------------------------------

Issue:
kubectl works for ubuntu but not Jenkins

Reason:
Jenkins using old kubeconfig

Fix:
Copy ~/.kube/config to /var/lib/jenkins/.kube

------------------------------------------------------------

Issue:
Trying kubectl create nodegroup

Reason:
NodeGroup is not a Kubernetes resource

Fix:
Use eksctl or AWS Console only

============================================================
IMPORTANT RULES (REMEMBER)
============================================================

- kubectl manages Kubernetes objects only
- Nodegroups are AWS infrastructure
- Nodegroups CANNOT be created with kubectl
- eksctl or AWS Console must be used
- Recreating cluster always changes EKS endpoint

============================================================
END OF FILE
============================================================
