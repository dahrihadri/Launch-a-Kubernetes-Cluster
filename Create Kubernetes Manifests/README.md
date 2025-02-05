# Kubernetes Deployment for Flask Backend

![201shots_so](https://github.com/user-attachments/assets/44819cc9-bb41-43cd-a187-e60d2f0e48a0)

This project walks you through deploying a Flask-based backend app to a Kubernetes cluster using Amazon Elastic Kubernetes Service (EKS). The app is containerized using Docker, and the Docker image is stored in Amazon Elastic Container Registry (ECR) for easy management and deployment.

## Table of Contents
- [Prerequisites](#prerequisites)
- [Steps Overview](#steps-overview)
- [Creating ECR Repository](#creating-ecr-repository)
- [Pushing Docker Image to ECR](#pushing-docker-image-to-ecr)
- [Creating Kubernetes Manifests](#creating-kubernetes-manifests)
- [Deploying the App to EKS](#deploying-the-app-to-eks)
- [Cleaning Up Resources](#cleaning-up-resources)
- [Additional Resources](#additional-resources)

---

## Prerequisites

Before you begin, make sure you have the following tools installed and configured:
- AWS CLI
- Docker
- `eksctl` for creating EKS clusters
- Kubernetes CLI (`kubectl`)
- An AWS account with appropriate permissions for ECR, EKS, and EC2.

---

# Setting up EC2 and EKS

![73shots_so (1)](https://github.com/user-attachments/assets/007425ee-cdbe-42f2-aa1a-04ccf52facc5)

In this step, we'll launch an EC2 instance and set up EKS (Elastic Kubernetes Service). The EC2 instance will serve as the command center, where we'll connect to AWS and manage Kubernetes.

---

## What is EKS?
Amazon Elastic Kubernetes Service (EKS) is a managed Kubernetes service that makes it easy to run Kubernetes clusters on AWS. 

Setting up Kubernetes from scratch can be complex and time-consuming, as you need to configure networking, scaling, and security settings. Amazon EKS automates these tasks, helping you integrate Kubernetes with other AWS services seamlessly.

If you're new to EKS, check out the first project of this Kubernetes series for a detailed explanation.

---

## In this step, we will:

- Launch and connect to an EC2 instance.
- Set up eksctl and configure AWS credentials.
- Create an EKS cluster.

---

### Step 1: Log into the AWS Management Console

1. Log into the AWS Management Console using your IAM Admin user credentials.

---

### Step 2: Launch and Connect to an EC2 Instance

1. **Head to the EC2 Console**  
   Go to the EC2 console and ensure you're in the AWS Region closest to you.

2. **Launch a New EC2 Instance**  
   Click on "Launch instances" to create a new EC2 instance.

3. **Configure the Instance**  
   - Name your instance `nextwork-eks-instance`.
   - Select **Amazon Linux 2023 AMI** as the Amazon Machine Image (AMI).
   - For the **Instance type**, select `t2.micro` (this is eligible for the free tier).
   - Under **Key pair**, select **Proceed without a key pair** (not recommended for production, but okay for this project).

4. **Network Settings**  
   - For simplicity, use the default security group for now.
   - **💡 Extra for PROs**: You can challenge yourself by editing the security group to restrict inbound rules, but for now, we’ll stick with the default security group to allow EC2 Instance Connect.

5. **Launch the Instance**  
   Click **Launch instance**.

6. **Connect to the EC2 Instance**  
   Once your instance is running, go to the EC2 console, select your `nextwork-eks-instance`, and click **Connect**.  
   Select **EC2 Instance Connect** and then click **Connect** again.

---

### Step 3: Install eksctl

`eksctl` is a command-line tool for managing Amazon EKS clusters. It simplifies the process of creating and managing EKS clusters.

1. **Install eksctl**  
   Run the following commands to install eksctl:

   ```bash
   curl --silent --location "https://github.com/weaveworks/eksctl/releases/latest/download/eksctl_$(uname -s)_amd64.tar.gz" | tar xz -C /tmp
   sudo mv -v /tmp/eksctl /usr/local/bin
   ```

2. **Verify eksctl Installation**  
   Check that `eksctl` is installed properly by running:

   ```bash
   eksctl version
   ```

   You should see the version number of `eksctl`.

   **🙋‍♀️ Stuck?** If you don’t see a version number, try running the installation commands again, and make sure there are no errors.

---

### Step 4: Create an IAM Role for the EC2 Instance

To allow your EC2 instance to interact with other AWS services, you need to assign it an IAM role.

1. **Navigate to IAM Console**  
   Go to the IAM console in AWS.

2. **Create a Role**  
   - Select **Roles** from the left sidebar.
   - Click **Create role**.
   - Choose **AWS service** as the trusted entity and select **EC2** as the use case.

3. **Assign Permissions**  
   - Select the **AdministratorAccess** policy.  
     **💡 Extra for PROs**: In a real-world scenario, you'd want to assign the minimum permissions necessary (principle of least privilege), but for now, we’ll use AdministratorAccess for simplicity.

4. **Name the Role**  
   Name the role `nextwork-eks-instance-role`.

5. **Create the Role**  
   Click **Create role** to finalize the creation.

---

### Step 5: Attach IAM Role to EC2 Instance

Now that the IAM role is created, we’ll attach it to our EC2 instance.

1. **Go to EC2 Console**  
   Return to the EC2 console and select the `nextwork-eks-instance`.

2. **Modify IAM Role**  
   - Select your instance, then click on **Actions** -> **Security** -> **Modify IAM role**.
   - Choose the `nextwork-eks-instance-role` from the dropdown.
   - Click **Update IAM role**.

---

### Step 6: Create an EKS Cluster

Now that your EC2 instance is properly configured, let’s create the EKS cluster.

1. **Run the Command to Create the EKS Cluster**  
   In your EC2 instance, run the following `eksctl` command to create the cluster. Be sure to replace `YOUR-REGION` with your AWS region code (e.g., `us-west-2`).

   ```bash
   eksctl create cluster      --name nextwork-eks-cluster      --nodegroup-name nextwork-nodegroup      --node-type t2.micro      --nodes 3      --nodes-min 1      --nodes-max 3      --version 1.31      --region YOUR-REGION
   ```

   This will take 15-20 minutes to complete, so feel free to grab a coffee while your cluster is being created.

---

### Step 7: Next Steps

While the cluster is being created, let’s move on to setting up the backend code you’ll be deploying to the EKS cluster.

---

## Additional Resources

- [AWS EKS Documentation](https://docs.aws.amazon.com/eks/latest/userguide/what-is-eks.html)
- [eksctl Documentation](https://eksctl.io/)
- [Amazon EC2 Documentation](https://docs.aws.amazon.com/ec2/index.html)

---

Happy building! 🎉


## Steps Overview

1. **Create an ECR Repository**: Store the Docker image in Amazon ECR for easy access by Kubernetes.
2. **Push Docker Image to ECR**: Build the Docker image and push it to ECR.
3. **Create Kubernetes Manifests**: Define how Kubernetes should deploy and expose the Flask app using deployment and service manifests.
4. **Deploy to EKS**: Deploy the app to a running EKS cluster using `kubectl`.
5. **Clean Up Resources**: Delete all resources to avoid incurring charges.

---



## Deploying the App to EKS

1. **Create EKS Cluster**  
Create a new EKS cluster with `eksctl`:

    ```bash
    eksctl create cluster --name nextwork-eks-cluster --region us-west-2 --nodegroup-name nextwork-nodes --node-type t3.micro --nodes 3
    ```

2. **Set Up kubectl**  
Update `kubectl` context to point to the newly created EKS cluster:

    ```bash
    aws eks --region us-west-2 update-kubeconfig --name nextwork-eks-cluster
    ```

3. **Apply Manifests**  
Apply both the deployment and service manifests using `kubectl`:

    ```bash
    kubectl apply -f flask-deployment.yaml
    kubectl apply -f flask-service.yaml
    ```

4. **Verify the Deployment**  
Check the status of your pods and services:

    ```bash
    kubectl get pods
    kubectl get svc
    ```

---

## Cleaning Up Resources

After you’ve finished the deployment, it’s important to clean up AWS resources to avoid incurring charges.

1. **Delete EKS Cluster**  
Use `eksctl` to delete your EKS cluster:

    ```bash
    eksctl delete cluster --name nextwork-eks-cluster --region us-west-2
    ```

2. **Terminate EC2 Instance**  
Terminate the EC2 instance used for your deployment:

    - Navigate to the EC2 console.
    - Select the instance and click on "Terminate."

3. **Delete ECR Repository**  
Delete the ECR repository:

    - Navigate to the ECR console.
    - Select the repository and click on "Delete."

---

## Additional Resources

- [AWS ECR Documentation](https://docs.aws.amazon.com/ecr/latest/userguide/what-is-ecr.html)
- [Docker Documentation](https://docs.docker.com/)
- [Kubernetes Documentation](https://kubernetes.io/docs/)

---

## License

This project is licensed under the MIT License - see the [LICENSE](LICENSE) file for details.
