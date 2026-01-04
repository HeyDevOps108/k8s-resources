AWS EKS COMPLETE SETUP – START TO END (PLAIN TEXT)

====================================================================
OVERVIEW
====================================================================

This file contains complete end-to-end steps to set up AWS EKS
for learning / non-production usage.

Covered topics:
- Tool installation
- IAM roles
- EKS cluster creation
- kubeconfig setup
- Nodegroup creation
- Namespace (sub-namespace style)
- Jenkins integration
- Common issues and fixes

Cluster Name      : cndevcorp5
AWS Region        : us-east-1

Naming Convention Used:
- Nodegroup        : <NodeGroup>
- Sub-namespace    : <Subnamespace>

NOTE:
<NodeGroup> is the NODEGROUP
<Subnamespace> is the SUB-NAMESPACE running on that nodegroup

====================================================================
PREREQUISITES
====================================================================

1. AWS account with required permissions
2. One EC2 instance (used as master / Jenkins server)
3. Internet access from EC2
4. IAM role attached to EC2 OR AWS access keys configured

====================================================================
INSTALL AWS CLI (v2)
====================================================================

curl "https://awscli.amazonaws.com/awscli-exe-linux-x86_64.zip" -o "awscliv2.zip"
sudo apt update
sudo apt install -y unzip
unzip awscliv2.zip
sudo ./aws/install

Verify:
aws --version

====================================================================
CONFIGURE AWS ACCESS
====================================================================

aws configure

Enter:
- AWS Access Key ID
- AWS Secret Access Key
- Default region: us-east-1
- Output format: json

Verify:
aws sts get-caller-identity

====================================================================
INSTALL kubectl
====================================================================

curl -LO https://storage.googleapis.com/kubernetes-release/release/`curl -s https://storage.googleapis.com/kubernetes-release/release/stable.txt`/bin/linux/amd64/kubectl
chmod +x kubectl
sudo mv kubectl /usr/local/bin/

Verify:
kubectl version --client

====================================================================
INSTALL eksctl
====================================================================

curl --silent --location "https://github.com/weaveworks/eksctl/releases/latest/download/eksctl_$(uname -s)_amd64.tar.gz" | tar xz -C /tmp
sudo mv /tmp/eksctl /usr/local/bin

Verify:
eksctl version

====================================================================
IAM ROLES (VERY IMPORTANT)
====================================================================

1. EKS CLUSTER SERVICE ROLE

IAM → Roles → Create role
Trusted entity: AWS service
Use case: EKS (Cluster)
Policy: AmazonEKSClusterPolicy
Example: eks-cluster-role-cndevcorp5

2. NODEGROUP IAM ROLE

IAM → Roles → Create role
Trusted entity: EC2
Policies:
- AmazonEKSWorkerNodePolicy
- AmazonEKS_CNI_Policy
- AmazonEC2ContainerRegistryReadOnly
Example: eks-nodegroup-role-cndevcorp5

3. JENKINS / MASTER IAM ROLE

Attach IAM role directly to EC2
Policies:
- AmazonEKSClusterPolicy
- AmazonEC2ContainerRegistryPowerUser
- AmazonEKSWorkerNodePolicy

====================================================================
CREATE EKS CLUSTER
====================================================================

Cluster name      : cndevcorp5
K8s version       : 1.29
Endpoint access   : Public + Private
Subnets           : ALL public + private
Observability     : Disabled

Wait until cluster becomes ACTIVE

====================================================================
CONFIGURE kubeconfig
====================================================================

aws eks update-kubeconfig --region us-east-1 --name cndevcorp5

Verify:
kubectl cluster-info
kubectl get nodes

====================================================================
CREATE NODEGROUP
====================================================================

NODEGROUP NAME: <NodeGroup>

eksctl create nodegroup \
  --cluster cndevcorp5 \
  --region us-east-1 \
  --name <NodeGroup> \
  --node-type t3.medium \
  --nodes 2 \
  --nodes-min 1 \
  --nodes-max 2 \
  --managed

Verify:
kubectl get nodes

====================================================================
CREATE SUB-NAMESPACE
====================================================================

SUB-NAMESPACE NAME: <Subnamespace>

apiVersion: v1
kind: Namespace
metadata:
  name: <Subnamespace>
  labels:
    env: preprod
    nodegroup: <NodeGroup>

Apply:
kubectl apply -f namespace.yaml

====================================================================
JENKINS kubeconfig FIX
====================================================================

sudo mkdir -p /var/lib/jenkins/.kube
sudo cp ~/.kube/config /var/lib/jenkins/.kube/config
sudo chown -R jenkins:jenkins /var/lib/jenkins/.kube

Verify:
sudo -u jenkins kubectl cluster-info

====================================================================
IMPORTANT RULES
====================================================================

- kubectl manages Kubernetes objects only
- Nodegroups are AWS infra
- Nodegroups CANNOT be created using kubectl
- Use eksctl or AWS Console
- EKS endpoint changes when cluster recreated

====================================================================
END OF FILE
====================================================================
