# Kubernetes Deployment Project

In this guide, I'll walk through the process of setting up a Kubernetes deployment for a backend application using AWS services like EKS, ECR, and EC2. By the end of this project, I will have deployed a backend application using Kubernetes and gained hands-on experience with containerization, AWS, and Kubernetes.

![163shots_so](https://github.com/user-attachments/assets/79d31949-c9f2-446e-a976-e085deb66d97)

---

## **Project Overview**
In this project, I will:
1. Set up an EC2 instance and an EKS cluster.
2. Clone a backend application from GitHub.
3. Build a Docker image of the backend.
4. Push the Docker image to Amazon ECR.
5. Troubleshoot common errors and explore the backend code.

---

## **Prerequisites**
- An AWS account.
- Basic knowledge of AWS services like EC2, VPC, CLI, and S3.
- Familiarity with Git, Docker, and Kubernetes.

---

## **Step 1: Set Up EC2 and EKS (Straight away to step 2 if you did not terminate resources created in project No. 1)**

### **1.1 Launch an EC2 Instance**
1. Log in to the AWS Management Console.
2. Navigate to the EC2 console.
3. Click **Launch Instances**.
   - Name: `nextwork-eks-instance`
   - AMI: **Amazon Linux 2023 AMI**
   - Instance Type: **t2.micro**
   - Network Settings: Use the default security group.
   - Key Pair: Proceed without a key pair.

     ![73shots_so](https://github.com/user-attachments/assets/d4fabbf4-8637-4914-98d4-8def86fd7fbc)

4. Click **Launch Instance**.

### **1.2 Connect to the EC2 Instance**
1. In the EC2 console, select your instance and click **Connect**.
2. Use **EC2 Instance Connect** to access the terminal.

   ![824shots_so](https://github.com/user-attachments/assets/73885506-1b40-424b-8a7d-8067a8662bad)

### **1.3 Install eksctl**
1. Run the following command to install `eksctl`:
   ```bash
   curl --silent --location "https://github.com/weaveworks/eksctl/releases/latest/download/eksctl_$(uname -s)_amd64.tar.gz" | tar xz -C /tmp
   sudo mv -v /tmp/eksctl /usr/local/bin
   ```
2. Verify the installation:
   ```bash
   eksctl version
   ```
   ![414shots_so](https://github.com/user-attachments/assets/f4546001-5bd7-4d39-a9dd-f71638cba69a)

### **1.4 Create an IAM Role for EC2**
1. Go to the IAM console.
2. Create a new role:
   - Trusted Entity: **AWS Service (EC2)**
   - Attach Policy: **AdministratorAccess**
   - Role Name: `nextwork-eks-instance-role`
3. Attach the role to your EC2 instance:
   - In the EC2 console, select your instance.
   - Go to **Actions > Security > Modify IAM Role**.
   - Attach the `nextwork-eks-instance-role`.

### **1.5 Create an EKS Cluster**
1. Run the following command to create an EKS cluster:
   ```bash
   eksctl create cluster \
     --name nextwork-eks-cluster \
     --nodegroup-name nextwork-nodegroup \
     --node-type t2.micro \
     --nodes 3 \
     --nodes-min 1 \
     --nodes-max 3 \
     --version 1.31 \
     --region YOUR-REGION
   ```
   Replace `YOUR-REGION` with your AWS region code (e.g., `us-west-2`).

---

## **Step 2: Pull the Backend Code from GitHub**

![678shots_so](https://github.com/user-attachments/assets/e2b4e9d8-a376-4624-8bd0-cade68c6305a)

### **2.1 Install Git**
1. Install Git on your EC2 instance:
   ```bash
   sudo dnf update
   sudo dnf install git -y
   ```
2. Verify the installation:
   ```bash
   git --version
   ```

### **2.2 Clone the Repository**
1. Clone the backend repository:
   ```bash
   git clone https://github.com/dahrihadri/Launch-a-Kubernetes-Cluster.git
   ```
2. Navigate to the cloned directory:
   ```bash
   cd Launch-a-Kubernetes-Cluster/Set\ Up\ Kubernetes\ Deployment/nextwork-flask-backend/
   ```

   ![618shots_so](https://github.com/user-attachments/assets/c8912a8e-9ca1-41ec-8c2d-556227159925)

---

## **Step 3: Build a Docker Image for the Backend**

![806shots_so](https://github.com/user-attachments/assets/f88261ab-85f0-44b8-bf8c-f6e54c81af55)

### **3.1 Install Docker**
1. Install Docker on your EC2 instance:
   ```bash
   sudo yum install -y docker
   ```
2. Start Docker:
   ```bash
   sudo service docker start
   ```

### **3.2 Add ec2-user to the Docker Group**
1. Add `ec2-user` to the Docker group:
   ```bash
   sudo usermod -a -G docker ec2-user
   ```
2. Restart your EC2 Instance Connect session.

### **3.3 Build the Docker Image**
1. Navigate to the backend directory:
   ```bash
   cd nextwork-flask-backend
   ```
2. Build the Docker image:
   ```bash
   docker build -t nextwork-flask-backend .
   ```

   ![573shots_so](https://github.com/user-attachments/assets/ffe17a67-1080-4a68-be1e-d634a80ba7b3)

---

## **Step 4: Push the Docker Image to Amazon ECR**

![837shots_so](https://github.com/user-attachments/assets/aa221afc-ec5e-4299-82d0-172dbc95d9b9)

### **4.1 Create an ECR Repository**
1. Run the following command to create an ECR repository:
   ```bash
   aws ecr create-repository \
     --repository-name nextwork-flask-backend \
     --image-scanning-configuration scanOnPush=true
   ```

   ![737shots_so](https://github.com/user-attachments/assets/b90758bc-bc20-4c53-953e-a0390d203652)

   ![938shots_so](https://github.com/user-attachments/assets/61d59ce5-e16f-47f8-b509-6f4735f5aadb)

### **4.2 Push the Docker Image to ECR**

   ![913shots_so](https://github.com/user-attachments/assets/1954fcc3-37d4-4e1c-9b06-2495abc44d73)

1. Authenticate Docker to your ECR repository:
   ```bash
   aws ecr get-login-password --region YOUR-REGION | docker login --username AWS --password-stdin YOUR-ACCOUNT-ID.dkr.ecr.YOUR-REGION.amazonaws.com
   ```
2. Tag and push the Docker image:
   ```bash
   docker tag nextwork-flask-backend:latest YOUR-ACCOUNT-ID.dkr.ecr.YOUR-REGION.amazonaws.com/nextwork-flask-backend:latest
   docker push YOUR-ACCOUNT-ID.dkr.ecr.YOUR-REGION.amazonaws.com/nextwork-flask-backend:latest
   ```

   ![779shots_so](https://github.com/user-attachments/assets/f113e796-53d9-4106-ba45-a83aee9e2836)

   ![689shots_so](https://github.com/user-attachments/assets/59e1ca80-5d36-4d3a-bdce-27e01b046da4)

---

## **Step 5: Explore the Backend Code (Secret Mission)**

1. Open the `nextwork-flask-backend` repository on GitHub.
2. Explore the following files:
   - `app.py`: The main Flask application.
   - `Dockerfile`: Instructions for building the Docker image.
   - `requirements.txt`: Python dependencies for the backend.

---

## **Step 6: Clean Up Resources**

### **6.1 Delete the EKS Cluster**
1. Run the following command to delete the EKS cluster:
   ```bash
   eksctl delete cluster --name nextwork-eks-cluster
   ```

### **6.2 Terminate the EC2 Instance**
1. Go to the EC2 console.
2. Select your instance and click **Instance State > Terminate**.

### **6.3 Delete the ECR Repository**
1. Go to the ECR console.
2. Select the `nextwork-flask-backend` repository and click **Delete**.

---

## **Conclusion**
Congratulations! We have successfully:
- Set up an EKS cluster.
- Cloned a backend application from GitHub.
- Built and pushed a Docker image to Amazon ECR.
- Explored the backend code.

---

In the next project of this Kubernetes series, you'll deploy the backend to see your work come to life in a live app!

## **Next Steps**
- Deploy the backend application using Kubernetes manifests.
- Explore advanced Kubernetes features like scaling and monitoring.

---

**Note:** Always clean up your AWS resources to avoid unnecessary charges.
