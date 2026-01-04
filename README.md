EKS INSTALLATION & INITIAL SETUP – STEP BY STEP (AWS EKS)

================================================
PREREQUISITES (ON MASTER / JENKINS SERVER)
================================================

1. Server must have:
   - Internet access
   - IAM role OR AWS access keys with EKS permissions

2. Check OS user:
   whoami

================================================
INSTALL AWS CLI (v2)
================================================

curl "https://awscli.amazonaws.com/awscli-exe-linux-x86_64.zip" -o "awscliv2.zip"
sudo apt update
sudo apt install -y unzip
unzip awscliv2.zip
sudo ./aws/install

Verify:
aws --version

================================================
CONFIGURE AWS ACCESS
================================================

aws configure
- Access Key
- Secret Key
- Region: us-east-1
- Output: json

Verify:
aws sts get-caller-identity

================================================
INSTALL kubectl
================================================

curl -LO https://storage.googleapis.com/kubernetes-release/release/`curl -s https://storage.googleapis.com/kubernetes-release/release/stable.txt`/bin/linux/amd64/kubectl
chmod +x kubectl
sudo mv kubectl /usr/local/bin/

Verify:
kubectl version --client

================================================
INSTALL eksctl
================================================

curl --silent --location "https://github.com/weaveworks/eksctl/releases/latest/download/eksctl_$(uname -s)_amd64.tar.gz" | tar xz -C /tmp
sudo mv /tmp/eksctl /usr/local/bin

Verify:
eksctl version

================================================
CREATE EKS CLUSTER (CONTROL PLANE)
================================================

(Using AWS Console – Recommended)

EKS → Create Cluster
- Cluster name: cndevcorp5
- Kubernetes version: 1.29
- Endpoint access: Public + Private
- Select all public + private subnets
- Observability: Disabled
- Add-ons: Default

Wait until status = ACTIVE

================================================
CONFIGURE kubeconfig
================================================

aws eks update-kubeconfig --region us-east-1 --name cndevcorp5

Verify:
kubectl cluster-info
kubectl get nodes   (should show: No resources found)

================================================
FIX SECURITY GROUP (IF kubectl TIMES OUT)
================================================

EKS → Cluster → Networking
- Find Cluster Security Group

Add inbound rule:
- HTTPS (443)
- Source: Security Group of master-tools-server

Ensure VPC settings:
- DNS resolution: Enabled
- DNS hostnames: Enabled

================================================
CREATE NODEGROUP (SIMPLE COMMAND)
================================================

eksctl create nodegroup \
  --cluster cndevcorp5 \
  --region us-east-1 \
  --name fsgbu-obcbcs-preprod \
  --node-type t3.medium \
  --nodes 2 \
  --nodes-min 1 \
  --nodes-max 2 \
  --managed

Verify:
kubectl get nodes

================================================
LABEL NODES (OPTIONAL – FOR NODEGROUP LOGIC)
================================================

kubectl label node <node-name> nodegroup=fsgbu-obcbcs-preprod env=preprod

Verify:
kubectl get nodes --show-labels

================================================
CREATE SUBNAMESPACE (SIMPLE)
================================================

apiVersion: v1
kind: Namespace
metadata:
  name: fsgbu-obcbcs-preprod--foundation
  labels:
    env: preprod
    nodegroup: fsgbu-obcbcs-preprod

Apply:
kubectl apply -f namespace.yaml

================================================
IMPORTANT JENKINS FIX (VERY IMPORTANT)
================================================

Whenever cluster is recreated, kubeconfig changes.

Copy kubeconfig to jenkins user:

sudo mkdir -p /var/lib/jenkins/.kube
sudo cp ~/.kube/config /var/lib/jenkins/.kube/config
sudo chown -R jenkins:jenkins /var/lib/jenkins/.kube

Verify:
sudo -u jenkins kubectl cluster-info

================================================
COMMON ISSUES & FIXES
================================================

1. kubectl openapi / DNS error
   → Old kubeconfig pointing to deleted cluster
   → Fix: aws eks update-kubeconfig + copy to jenkins

2. kubectl timeout to 10.x.x.x
   → Private endpoint only
   → Fix: Enable public endpoint or SG rule

3. kubectl apply fails in Jenkins but works in shell
   → Jenkins using old kubeconfig

================================================
IMPORTANT RULES
================================================

- kubectl = Kubernetes objects only
- eksctl / AWS CLI = Infrastructure (nodegroups, EC2)
- NodeGroup CANNOT be created with kubectl
- Namespace CAN be created with kubectl

================================================
END OF FILE
================================================
