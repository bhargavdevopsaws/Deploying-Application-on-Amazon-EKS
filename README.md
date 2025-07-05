Kubernetes End to End Project on EKS(Amazon Kubernetes Service)

Prerequisites

> kubectl – A command line tool for working with Kubernetes clusters. For more information, see Installing or updating kubectl.

> eksctl – A command line tool for working with EKS clusters that automates many individual tasks. For more information, see Installing or updating.

> AWS CLI – A command line tool for working with AWS services, including Amazon EKS. For more information, see Installing, updating, and uninstalling the AWS CLI in the AWS Command Line Interface User Guide.

After installing the AWS CLI, we recommend that you also configure it. For more information, see Quick configuration with aws configure in the AWS Command Line Interface User Guide.

Project Title: Deploying 2048 Game App on Amazon EKS

Project Description:

A Kubernetes End-to-End (E2E) project for deploying a 2048 game app on Amazon Elastic Kubernetes Service (EKS) involves setting up, deploying, and managing the popular 2048 game application on a Kubernetes cluster running on AWS EKS. This project aims to demonstrate how to containerize a web application, deploy it on EKS, manage the cluster, and expose the application to users.

Containerization:
I began by containerizing the 2048 game using Docker. This involved creating a Dockerfile to define the application's runtime environment and dependencies, ultimately resulting in a Docker image ready for deployment.

Amazon EKS Setup:
I set up an Amazon EKS cluster, configuring the required resources and network settings using AWS services. This step included authentication and permissions setup to interact with the EKS cluster.

Deployment:
The containerized 2048 game was deployed on the EKS cluster using Kubernetes. I defined Kubernetes deployment and service YAML files to ensure the application's efficient management and availability.

Scaling and Management:
I explored Kubernetes's scaling capabilities, adjusting the number of application replicas based on demand. This ensured the game could handle varying levels of user traffic seamlessly.

Application Exposure:
To make the 2048 game accessible to users, I created a Kubernetes service to expose it securely over the internet. Additionally, I could have implemented an Ingress controller for more advanced routing

Create IAM Roles

You need two IAM roles:

Cluster Role

1.	Go to IAM > Roles > Create Role
2.	Choose EKS > EKS - Cluster
3.	Attach AmazonEKSClusterPolicy
4.	Name: eks-cluster-role

 
Create another IAM role 'eks-node-grp-role' with 3 policies attached: 

(Allows EC2 instances to call AWS services on your behalf.)

Node Group Role

1.	Create another role
2.	Choose EKS > EKS - Nodegroup
3.	Attach the following policies:
o	AmazonEKSWorkerNodePolicy
o	AmazonEC2ContainerRegistryReadOnly
o	AmazonEKS_CNI_Policy
4.	Name: eks-node-grp-role

Launch EC2 Ubuntu 22.04 Instance (if not already done)

You can launch it from the AWS Console:
•	AMI: Ubuntu 22.04 
•	Instance Type: t2.medium or t3.medium (recommended for this task)
•	Enable auto-assign public IP
•	Add a key pair (e.g.,awsdevops)
 
Connect to Your EC2 Ubuntu Instance

From your local terminal:

•	sudo apt update

•	sudo apt upgrade -y

•	curl "https://awscli.amazonaws.com/awscli-exe-linux-x86_64.zip" -o "awscliv2.zip"

•	sudo apt install zip

•	sudo apt install unzip -y

•	unzip awscliv2.zip

•	sudo ./aws/install

•	aws –version


Configure AWS CLI: 

Install kubectl
•	curl -s https://dl.k8s.io/release/stable.txt

Replace v1.30.1 with the version you got (or use the other current version)
•	curl -LO https://dl.k8s.io/release/v1.30.1/bin/linux/amd64/kubectl

•	chmod +x kubectl

•	sudo mv kubectl /usr/local/bin/

•	kubectl version –client

 
Install eksctl
•	curl –location "https://github.com/weaveworks/eksctl/releases/latest/download/eksctl_$(uname -s)_amd64.tar.gz" | tar xz

•	sudo mv eksctl /usr/local/bin

•	eksctl version
 
Create EKS Cluster:

eksctl create cluster \
--name 2048-eks-cluster \
--version 1.29 \
--region ap-south-1 \
--nodegroup-name 2048-node-group \
--node-type t3.medium \
--nodes 1 \
--nodes-min 1 \
--nodes-max 2 \
--node-volume-size 20 \
--managed \
--with-oidc \
--ssh-access \
--ssh-public-key awsdevops\
--full-ecr-access \
--asg-access \
--alb-ingress-access

Note:
•	Replace the region with your desired location you choose.
•	Replace nodegroup-name according to your desired name.
•	Replace (awsdevops) with the exact name of your EC2 key pair created in the AWS Console.


Verify Cluster Creation

Cluster creation takes around 15 minutes. Once done:

•	aws eks --region us-east-1 describe-cluster --name 2048-eks-cluster --query "cluster.status"
 
 
•	eksctl get cluster --region us-east-1
 
 

Authentication & Connection:
Update kubeconfig
•	aws eks update-kubeconfig --region ap-south-1 --name bhargav-eks-cluster

Verify Cluster
•	kubectl get nodes
You should see a node (worker node) in Ready state. 
 

Create and Deploy Pod:

Create the YAML Manifest File (2048-pod.yaml):

Note: Use any text editor to create the YAML file. If using AWS CloudShell or a Linux terminal:

•	nano 2048-pod.yaml

Paste the following YAML code inside the file:

apiVersion: v1
kind: Pod
metadata:
  name: 2048-pod
  labels:
    app: 2048-ws
spec:
  containers:
    - name: 2048-container
      image: blackicebird/2048
      ports:
        - containerPort: 80

 
•	kubectl apply -f 2048-pod.yaml
•	kubectl get pods
 
Expose Pod with LoadBalancer:
Create Service (mygame-svc.yaml):
•	nano mygame-svc.yaml

apiVersion: v1
kind: Service
metadata:
  name: mygame-svc
spec:
  selector:
    app: 2048-ws
  ports:
    - protocol: TCP
      port: 80
      targetPort: 80
  type: LoadBalancer
 
•	kubectl apply -f mygame-svc.yaml
•	kubectl get svc
•	kubectl describe svc mygame-svc
 
Access the Application:
•	Find the EXTERNAL-IP / ELB DNS Name:

•	When you run the kubectl get svc command copy the External-IP of the load balancer.

•	Your EXTERNAL-IP is the ELB DNS name assigned by AWS may look like this.

**ad3fa4bb46e21469bbc3ddfd7852101c 158020008.ap south 1.elb.amazonaws.com**

Access via Browser

Open this URL in your browser:

http:// ad3fa4bb46e21469bbc3ddfd7852101c 158020008.ap south 1.elb.amazonaws.com
 


