# Project Name
EKS-based Kubernetes Deployment (Learning / Non-Prod)

==================================================
1. PROJECT OVERVIEW
==================================================

This project demonstrates a real-world Kubernetes setup on AWS EKS.
It covers cluster creation, nodegroups, namespaces, application deployment,
and CI/CD integration using Jenkins.

This setup is for learning and non-production purposes.

--------------------------------------------------

2. ARCHITECTURE OVERVIEW
--------------------------------------------------

- AWS EKS Cluster
- Managed Node Group (EC2)
- Kubernetes Namespaces (environment-based)
- Application deployed as Kubernetes Deployment
- CI/CD via Jenkins
- Future: Ingress + ALB

--------------------------------------------------

3. PREREQUISITES
--------------------------------------------------

- AWS Account
- IAM permissions for EKS, EC2, VPC
- EC2 server for:
  - Jenkins
  - kubectl
  - aws cli
  - eksctl
- Internet access from EC2

--------------------------------------------------

4. TOOLS & VERSIONS
--------------------------------------------------

- AWS CLI v2
- kubectl
- eksctl
- Jenkins
- Docker
- Kubernetes (EKS v1.29)

--------------------------------------------------

5. AWS EKS CLUSTER SETUP
--------------------------------------------------

Cluster Name: cndevcorp5
Region: us-east-1

Steps:
1. Create EKS cluster using AWS Console
2. Enable Public + Private endpoint access
3. Select all public and private subnets
4. Disable observability (for learning)
5. Wait until cluster status is ACTIVE

--------------------------------------------------

6. KUBECONFIG SETUP
--------------------------------------------------

Command used:
aws eks update-kubeconfig --region us-east-1 --name cndevcorp5

Verification:
kubectl cluster-info
kubectl get nodes

--------------------------------------------------

7. NODEGROUP SETUP
--------------------------------------------------

Nodegroup Name: fsgbu-obcbcs-preprod
Instance Type: t3.medium
Desired Nodes: 2

Command used:
eksctl create nodegroup \
  --cluster cndevcorp5 \
  --region us-east-1 \
  --name fsgbu-obcbcs-preprod \
  --node-type t3.medium \
  --nodes 2 \
  --nodes-min 1 \
  --nodes-max 2 \
  --managed

--------------------------------------------------

8. NODE LABELING (OPTIONAL)
--------------------------------------------------

Nodes are labeled for logical grouping.

Example:
nodegroup=fsgbu-obcbcs-preprod
env=preprod

Used for nodeSelector in deployments.

--------------------------------------------------

9. NAMESPACE STRUCTURE
--------------------------------------------------

Namespaces follow environment-based naming.

Example:
fsgbu-obcbcs-preprod--foundation

Namespace YAML:
apiVersion: v1
kind: Namespace
metadata:
  name: fsgbu-obcbcs-preprod--foundation
  labels:
    env: preprod
    nodegroup: fsgbu-obcbcs-preprod

--------------------------------------------------

10. APPLICATION DEPLOYMENT
--------------------------------------------------

- Application runs inside Kubernetes as a Deployment
- Service type: ClusterIP
- Exposed later using Ingress

Key files:
- deployment.yaml
- service.yaml

--------------------------------------------------

11. CI/CD WITH JENKINS
--------------------------------------------------

Jenkins is used to:
- Build Docker image
- Push image to ECR
- Apply Kubernetes manifests using kubectl

Important:
Whenever EKS cluster is recreated, kubeconfig must be copied
from ubuntu user to jenkins user.

--------------------------------------------------

12. COMMON ISSUES & FIXES
--------------------------------------------------

Issue: kubectl openapi / DNS error
Fix: kubeconfig was pointing to old EKS endpoint

Issue: kubectl works for ubuntu but not jenkins
Fix: copy ~/.kube/config to /var/lib/jenkins/.kube

Issue: kubectl timeout to 10.x.x.x
Fix: enable public endpoint or add SG rule

--------------------------------------------------

13. COST MANAGEMENT
--------------------------------------------------

- Nodegroups are scaled down or deleted when not in use
- LoadBalancers are removed after testing
- EKS control plane cost is minimal

--------------------------------------------------

14. FUTURE ENHANCEMENTS
--------------------------------------------------

- Ingress with AWS ALB
- Path-based routing
- HPA (Horizontal Pod Autoscaler)
- Helm charts
- Monitoring and logging

--------------------------------------------------

15. NOTES
--------------------------------------------------

- kubectl manages Kubernetes resources only
- Nodegroups cannot be created using kubectl
- eksctl or AWS Console is required for infra

==================================================
END OF README
==================================================
