# Prometheus-and-Grafana-Dashboard-on-EKS-Cluster-using-Helm-Chart
Prometheus and Grafana Dashboard on EKS Cluster using Helm Chart

## High level Tasks 
1. Create AWS Ec2 Instance and install the required softwares
2. Create AWS EKS cluster
3. Installing the Kubernetes Metrics Server
4. Install Prometheus
5. Create IAM OIDC Provider
6. Install Grafana
7. Import Grafana dashboard from Grafana Labs
8. Deploy a Node.js application and monitor it on Grafana

   
## 1. Create AWS Ec2 Instance and install the required softwares

1. Login to an AWS account using a user with admin privileges
2. Move to the EC2 console. Click Launch Instance.
3. Select AMIs as Ubuntu and select Instance Type as t2.medium. Create new Key Pair and Create a new Security Group with traffic allowed from ssh, http and https.

```
Install the required softwares on the server -

1. Install AWS CLI and Configure >>

curl "https://awscli.amazonaws.com/awscli-exe-linux-x86_64.zip" -o "awscliv2.zip" 
sudo apt install unzip
unzip awscliv2.zip 
sudo ./aws/install

## Post AWS CLI installation run the below mentioned coomand to setup AWS user command line integration mode (CLI).
AWS ACCESS KEY, SECRET KEY you will can generate from IAM >> User >> Select User >> Generate Key 

aws configure 
AWS Access Key ID [None]: AKIAIOSFODNN7EXAMPLE        ## Input your user ID
AWS Secret Access Key [None]: wJalrXUtnFEMI/K7MDENG/bPxRfiCYEXAMPLEKEY    ## Input your user secret key 
Default region name [None]: us-west-2   ## Input your AWS Region
Default output format [None]: json 

2. Install and Setup Kubectl >>

curl -LO "https://storage.googleapis.com/kubernetes-release/release/$(curl -s https://storage.googleapis.com/kubernetes-release/release/stable.txt)/bin/linux/amd64/kubectl"
chmod +x ./kubectl
sudo mv ./kubectl /usr/local/bin
kubectl version

3. Install and Setup eksctl >>

curl --silent --location "https://github.com/weaveworks/eksctl/releases/latest/download/eksctl_$(uname -s)_amd64.tar.gz" | tar xz -C /tmp
sudo mv /tmp/eksctl /usr/local/bin
eksctl version

4. Install Helm chart >>
curl -fsSL -o get_helm.sh https://raw.githubusercontent.com/helm/helm/main/scripts/get-helm-3
chmod 700 get_helm.sh
./get_helm.sh

```

## 2. Create AWS EKS cluster

we are going to create Amazon EKS cluster using eksctl

```
You need the following in order to run the eksctl command >>

eksctl create cluster --name eks2\ ## it will craete the EKS cluster 
--version 1.24 \  
--region us-east-1 \
--nodegroup-name worker-nodes
--node-type t2.large \
--nodes 2 \
--nodes-min 2 \
--nodes-max 3\

aws eks update-kubeconfig --name eks4
eksctl get all -A   ## Run this command to check the EKS cluster post creation 

```

You can go back to your AWS dashboard and look for Elastic Kubernetes Service -> Clusters

![image](https://github.com/anand40090/Prometheus-and-Grafana-Dashboard-on-EKS-Cluster-using-Helm-Chart/assets/32446706/9345f4dd-c9e7-437d-aecc-163e7187fee5)
