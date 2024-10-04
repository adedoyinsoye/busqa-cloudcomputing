# busqa-cloudcomputing
Cloud Computing Project that provisions an EKS Cluster on AWS using Terraform. 


This repository contains all the necessary files to provision an EKS cluster using Terraform and deploy both the WordPress and Voting App applications using Kubernetes. The applications are configured to scale automatically based on CPU/memory usage and can be accessed via simulated domain names (vote.busyqa.com, results.busyqa.com, and wordpress.busyqa.com) by configuring the local /etc/hosts file.

# Prerequisites

Before you begin, ensure you have the following tools installed:

Terraform (v1.0 or later)
kubectl (compatible with Kubernetes version 1.29)
AWS CLI (configured with access to your AWS account)
Helm (for installing the NGINX Ingress controller)
Docker (to build Docker images if needed)
You will also need an AWS account with the appropriate permissions to create EKS clusters, VPCs, and other related resources.

# Setup Instructions
Clone the repository
First, clone this repository to your local machine:

git clone https://github.com/your-username/busyqa-cloudcomputing.git

Change your directory to the cloned repo's directory: 

cd busyqa-cloudcomputing 

# Provisioning the EKS Cluster
Initialize and apply Terraform configuration
To provision the EKS cluster, navigate to the terraform/ directory and initialize Terraform:

cd terraform/
terraform init

Next, apply the Terraform configuration to create the necessary AWS resources:

terraform apply

This command will provision:

A custom VPC with both private and public subnets.
An EKS cluster with worker nodes in private subnets (without public IPs).
A NAT Gateway for outbound internet access from the private worker nodes.

Install the NGINX Ingress Controller
After provisioning the cluster, install the NGINX Ingress Controller to manage HTTP routing:


helm repo add ingress-nginx https://kubernetes.github.io/ingress-nginx

helm install nginx-ingress ingress-nginx/ingress-nginx --set controller.publishService.enabled=true

# Deploying Applications
Once the EKS cluster is provisioned, deploy the WordPress and Voting App applications using Kubernetes manifests.

Deploy WordPress
Go to the manifests/wordpress/ directory and apply the WordPress manifests:


cd ../manifests/wordpress/
kubectl apply -f .

This deploys:

A WordPress deployment with 3 replicas.
A MySQL database deployment with persistent storage (PVCs).
A LoadBalancer service to expose WordPress.

Deploy the Voting App
Go to the manifests/voting-app/ directory and apply the Voting App manifests:


cd ../voting-app/
kubectl apply -f .

This deploys the following components:

Vote Frontend: The frontend service for the Voting App (Python).
Result Backend: The backend service for displaying results (Node.js).
Worker: The worker service responsible for transferring votes from Redis to PostgreSQL.
Redis: A Redis instance for caching votes.
PostgreSQL: A database for storing vote results.

# Simulating DNS with /etc/hosts
Since you may not have access to real domain names, you can simulate domain names on your local machine by editing the /etc/hosts file.

Get the LoadBalancer External IPs
Run the following command to get the EXTERNAL-IP of the LoadBalancer services:


kubectl get services

Edit /etc/hosts
Open your local machine's /etc/hosts file for editing:

Linux/macOS:


sudo nano /etc/hosts

Windows: Open C:\Windows\System32\drivers\etc\hosts in Notepad (run as Administrator).

Add the following entries (replace <EXTERNAL-IP> with the LoadBalancer DNS names or IP you retrieved earlier):

Example code

a37bb4b9550ae407eaf92379fa1cc0bd-1247859352.ca-central-1.elb.amazonaws.com vote.busyqa.com
aaf5ba59d31a44dd695661ca8b11befb-331786969.ca-central-1.elb.amazonaws.com results.busyqa.com
a117495e7a9934fc0b8fdfc8e27b8537-1679691831.ca-central-1.elb.amazonaws.com wordpress.busyqa.com

Access the Applications
After modifying the /etc/hosts file, you can access the applications via the following URLs:

Voting App Frontend: http://vote.busyqa.com
Result Backend: http://results.busyqa.com
WordPress: http://wordpress.busyqa.com


# Clean-up Instructions

Once testing is complete, clean up the resources to avoid unnecessary costs.

Destroy Terraform Resources
Navigate to the terraform/ directory and run the following command to destroy the EKS cluster and other AWS resources:

terraform destroy

Delete the CPU Stress Test Job
Delete the load generator job if it was applied:


kubectl delete job cpu-stress-test

This will stop the artificial CPU load.

