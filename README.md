# aws-kubernetes---CLI

# 🚀 Deploying Amazon EKS with EC2 Node Groups & Standalone EC2 Instances

This project demonstrates how to deploy a scalable **Amazon EKS (Elastic Kubernetes Service)** cluster using a combination of EC2 node groups and a separate Amazon Linux 2 instance. It includes IAM configuration, CLI setup, cluster creation, troubleshooting steps, and workload validation via Kubernetes commands.

## 📦 Tools & Technologies Used
- Amazon EKS, EC2
- IAM Roles & Policies
- AWS CLI, kubectl, eksctl
- Kubernetes (1.33), Amazon Linux 2 (t2.micro)

## ✅ Summary of Deployment Steps
### 1. IAM Role Setup
- **Cluster IAM Role Policies**:
  - `AmazonEKSClusterPolicy`
  - `AmazonEKSServicePolicy`
- **Node IAM Role Policies**:
  - `AmazonEKSWorkerNodePolicy`
  - `AmazonEKS_CNI_Policy`
  - `AmazonEC2ContainerRegistryReadOnly`
  - Optional: BlockStorage, Compute, LoadBalancing, Networking policies

### 2. CLI Tool Installation
```bash
# AWS CLI
curl "https://awscli.amazonaws.com/awscli-exe-linux-x86_64.zip" -o "awscliv2.zip"
unzip awscliv2.zip
sudo ./aws/install

# eksctl - See official docs
# kubectl
curl -O https://s3.us-west-2.amazonaws.com/amazon-eks/1.33.0/2025-05-01/bin/linux/amd64/kubectl
chmod +x ./kubectl
mkdir -p $HOME/bin && cp ./kubectl $HOME/bin/kubectl && export PATH=$HOME/bin:$PATH
echo 'export PATH=$HOME/bin:$PATH' >> ~/.bashrc
kubectl version --client
```

### 3. EKS Cluster Creation
```bash
aws eks --region us-east-1 update-kubeconfig --name EKSClusterJLP
aws eks describe-cluster --name EKSClusterJLP --region us-east-1 --query cluster.status
```

### 4. EC2 Launch (Separate from Node Group)
- Amazon Linux 2 AMI
- Instance type: `t2.micro`
- Key pair: `EKSCluster`
- Security group: Allow SSH/HTTP

### 5. Node Group Configuration
- Name: `myworkernodeJLP`
- Role: `AmazonEKSAutoNodeRole`
- Disk: 20 GB
- Remote access enabled via EC2 key
- Subnets: 3 selected

### 6. Kubernetes Verification
```bash
kubectl get svc
kubectl get pods --all-namespaces
kubectl get nodes --watch
kubectl create deployment nginx --image=nginx
kubectl logs nginx-<pod-id>
```

### 7. Troubleshooting Tips
```bash
# Delete and recreate node group if failed
aws eks delete-nodegroup --cluster-name EKSClusterJLP --nodegroup-name myworkernodeJLP --region us-east-1

# Check node group status
aws eks describe-nodegroup --cluster-name EKSClusterJLP --nodegroup-name myworkernodeJLP --region us-east-1 --query nodegroup.status

# Diagnose nodes stuck in NotReady
sudo journalctl -u kubelet -f
aws ec2 reboot-instances --instance-ids <instance-id>
```

## 🌟 Outcomes
Successfully deployed a working EKS cluster with:
- Proper IAM permissions
- Valid node registration
- Workload execution via Kubernetes
- Thorough troubleshooting

## 💡 Lessons Learned
This project highlights the critical importance of:
- Role-based access controls (RBAC)
- Secure networking practices
- Iterative debugging in cloud-native environments
