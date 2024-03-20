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

Output :-

![image](https://github.com/anand40090/Prometheus-and-Grafana-Dashboard-on-EKS-Cluster-using-Helm-Chart/assets/32446706/9345f4dd-c9e7-437d-aecc-163e7187fee5)

## 3. Installing the Kubernetes Metrics Server

Install the Kubernetes Metrics server onto the Kubernetes cluster so that Prometheus can collect the performance metrics of Kubernetes.

```
Deploy the Metrics Server with the following command >>
kubectl apply -f https://github.com/kubernetes-sigs/metrics-server/releases/latest/download/components.yaml

Verify that the metrics-server deployment is running the desired number of pods with the following command >>
kubectl get deployment metrics-server -n kube-system

```
Output :-

![image](https://github.com/anand40090/Prometheus-and-Grafana-Dashboard-on-EKS-Cluster-using-Helm-Chart/assets/32446706/39285c1d-c4d2-4acd-9946-239d285964e5)

## 4. Install Prometheus

Now install the Prometheus using the helm chart.

```
Add Prometheus helm chart repository >>
helm repo add prometheus-community https://prometheus-community.github.io/helm-charts

Update the helm chart repository >>
helm repo update
helm repo list

Create prometheus namespace >>
kubectl create namespace prometheus

Install Prometheus >>

 helm install prometheus prometheus-community/prometheus \
    --namespace prometheus \
    --set alertmanager.persistentVolume.storageClass="gp2" \
    --set server.persistentVolume.storageClass="gp2"

```
Output :-

![image](https://github.com/anand40090/Prometheus-and-Grafana-Dashboard-on-EKS-Cluster-using-Helm-Chart/assets/32446706/8f4726bc-cbe2-4e9d-8375-7b00c1d2c50d)


## 5. Create IAM OIDC Provider

Your cluster has an OpenID Connect (OIDC) issuer URL associated with it. 
To use AWS Identity and Access Management (IAM) roles for service accounts, 
an IAM OIDC provider must exist for your cluster's OIDC issuer URL.

```
Add IAM Role using eksctl with your cluster name >>
eksctl create iamserviceaccount \
  --name ebs-csi-controller-sa \
  --namespace kube-system \
  --cluster eks4 \
  --attach-policy-arn arn:aws:iam::aws:policy/service-role/AmazonEBSCSIDriverPolicy \
  --approve \
  --role-only \
  --role-name AmazonEKS_EBS_CSI_DriverRole

Then add EBS CSI to eks by running the following command,Enter your account ID and cluster name >>

eksctl create addon --name aws-ebs-csi-driver --cluster eks4 --service-account-role-arn arn:aws:iam::xxxxxxxxx:role/AmazonEKS_EBS_CSI_DriverRole --force

```

Finally, all pods are running now 

Output :-

![image](https://github.com/anand40090/Prometheus-and-Grafana-Dashboard-on-EKS-Cluster-using-Helm-Chart/assets/32446706/e8488719-0fa3-4e38-a873-0f8d0e90da84)

View the Prometheus dashboard by forwarding the deployment ports

Output :-

![image](https://github.com/anand40090/Prometheus-and-Grafana-Dashboard-on-EKS-Cluster-using-Helm-Chart/assets/32446706/12c3ff47-1f76-483a-81a8-091ff557267f)


## 6. Install Grafana

Add the Grafana helm chart repository. Later, Update the helm chart repository.

```

helm repo add grafana https://grafana.github.io/helm-charts 
helm repo update

```

Now we need to create a Prometheus data source so that Grafana can access the Kubernetes metrics. 
Create a yaml file prometheus-datasource.yaml and save the following data source configuration into it -

```
datasources:
  datasources.yaml:
    apiVersion: 1
    datasources:
    - name: Prometheus
      type: prometheus
      url: http://prometheus-server.prometheus.svc.cluster.local
      access: proxy
      isDefault: true

```

Create a namespace grafana

```
kubectl create namespace grafana

```

Install the Grafana

```
helm install grafana grafana/grafana \
    --namespace grafana \
    --set persistence.storageClassName="gp2" \
    --set persistence.enabled=true \
    --set adminPassword='EKS!sAWSome' \    ## Not this password it will require to login Gafana Console 
    --values prometheus-datasource.yaml \
    --set service.type=LoadBalancer
```
Output :-

![image](https://github.com/anand40090/Prometheus-and-Grafana-Dashboard-on-EKS-Cluster-using-Helm-Chart/assets/32446706/58249f2a-f866-46d5-af13-c82ad7642245)


Verify the Grafana installation by using the following kubectl command -

Output :-

![image](https://github.com/anand40090/Prometheus-and-Grafana-Dashboard-on-EKS-Cluster-using-Helm-Chart/assets/32446706/af0d923a-7a44-4ea8-9bc5-259ccd8d7e57)


Copy External IP address and open it in the browser -

Password you mentioned as EKS!sAWSome while creating Grafana

Output :-

![image](https://github.com/anand40090/Prometheus-and-Grafana-Dashboard-on-EKS-Cluster-using-Helm-Chart/assets/32446706/8a92e41b-95cb-4883-a446-9aa4676973d6)


## 7. Import Grafana dashboard from Grafana Labs

Now we have set up everything in terms of Prometheus and Grafana. 
For the custom Grafana Dashboard, we are going to use the open source grafana dashboard. 
For this session, I am going to import a dashboard 6417

Output :-

![image](https://github.com/anand40090/Prometheus-and-Grafana-Dashboard-on-EKS-Cluster-using-Helm-Chart/assets/32446706/7a80baae-a245-4b30-afb0-2bcaf8856b99)

Load and select the source as Prometheus

Output :-

![image](https://github.com/anand40090/Prometheus-and-Grafana-Dashboard-on-EKS-Cluster-using-Helm-Chart/assets/32446706/e8b39c47-d8be-4c5c-a98b-52ad82816e8f)

Import it-

Output :-

![image](https://github.com/anand40090/Prometheus-and-Grafana-Dashboard-on-EKS-Cluster-using-Helm-Chart/assets/32446706/24d57bdd-c0e1-448f-a94f-c7d0a4206a3a)


## 8. Deploy a Node.js application and monitor it on Grafana

To make use of Grafana dashboard, we will deploy Node.js application on Kubernetes. 
Download deployment.yml file from the below repository.

https://github.com/sunitabachhav2007/node-todo-cicd

To deploy the Node.js application on kubernetes cluster user the following kubectl command. 
Verify the deployment by running the following kubectl command

```
kubectl apply -f deployment.yml
kubectl get deployment
kubectl get pods
kubectl get service  ## To check the external IP of the EKS servcie it will need to browse the dashboard

```

Output :-

![image](https://github.com/anand40090/Prometheus-and-Grafana-Dashboard-on-EKS-Cluster-using-Helm-Chart/assets/32446706/9c1eb76c-ffbc-47da-a27f-f81df8653bde)

![image](https://github.com/anand40090/Prometheus-and-Grafana-Dashboard-on-EKS-Cluster-using-Helm-Chart/assets/32446706/a4a5d6d2-5226-4f0e-a734-ab022f6dc6d8)

The Node.js Application is running successfully 

Output :-

![image](https://github.com/anand40090/Prometheus-and-Grafana-Dashboard-on-EKS-Cluster-using-Helm-Chart/assets/32446706/bec9d93b-3c19-40a2-ba53-c11f6f9a3bb2)

Refresh the Grafana dashboard to verify the deployment

Output :-

![image](https://github.com/anand40090/Prometheus-and-Grafana-Dashboard-on-EKS-Cluster-using-Helm-Chart/assets/32446706/d8c78c0d-fa1c-4a42-9135-df1d84adbf3c)

