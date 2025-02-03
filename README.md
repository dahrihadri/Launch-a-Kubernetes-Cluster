# Launch a Kubernetes Cluster

This project guides you through deploying a Kubernetes cluster using Amazon Elastic Kubernetes Service (EKS). It's designed for beginners to gain hands-on experience with Kubernetes, AWS EKS, EC2, CloudFormation, and IAM.

## Table of Contents
1. [Introduction](#introduction)
2. [Prerequisites](#prerequisites)
3. [Step-by-Step Guide](#step-by-step-guide)
   - [Step 1: Launch and Connect to an EC2 Instance](#step-1-launch-and-connect-to-an-ec2-instance)
   - [Step 2: Install `eksctl` and Attempt to Create a Cluster](#step-2-install-eksctl-and-attempt-to-create-a-cluster)
   - [Step 3: Create an IAM Role and Launch the EKS Cluster](#step-3-create-an-iam-role-and-launch-the-eks-cluster)
   - [Step 4: Monitor Cluster Creation with CloudFormation](#step-4-monitor-cluster-creation-with-cloudformation)
   - [Step 5: Access the EKS Cluster via AWS Management Console](#step-5-access-the-eks-cluster-via-aws-management-console)
   - [Secret Mission: Test Cluster Resilience](#secret-mission-test-cluster-resilience)
4. [Cleanup](#cleanup)
5. [Conclusion](#conclusion)

---

## Introduction

In this project, you will:
- Launch and connect to an EC2 instance.
- Create a Kubernetes cluster using Amazon EKS.
- Monitor the cluster creation process using AWS CloudFormation.
- Access and manage the cluster via the AWS Management Console.
- Test the resilience of your Kubernetes cluster by deleting nodes and observing their automatic recovery.

This project is part of a series aimed at taking you from zero experience to deploying a containerized application using Kubernetes.

---

## Prerequisites

Before starting, ensure you have:
1. An **AWS account** ([create one here](https://aws.amazon.com/)).
2. Basic familiarity with AWS services like EC2, VPC, and IAM.
3. A terminal or SSH client to connect to your EC2 instance.

---

## Step-by-Step Guide

### Step 1: Launch and Connect to an EC2 Instance

1. **Log in to AWS Management Console** as your IAM Admin user.
2. Navigate to **EC2** and select **Instances**.
3. Click **Launch Instances**.
   - Name: `nextwork-eks-instance`
   - AMI: **Amazon Linux 2023 AMI**
   - Instance Type: **t2.micro**
   - Key Pair: Proceed without a key pair (we'll use EC2 Instance Connect).
   - Network Settings: Use the default security group.
4. Click **Launch Instance**.
5. Connect to the instance using **EC2 Instance Connect**:
   - Select the instance, click **Connect**, and choose **EC2 Instance Connect**.
   - A terminal will open in your browser.

---

### Step 2: Install `eksctl` and Attempt to Create a Cluster

1. **Install `eksctl`**:
   ```bash
   curl --silent --location "https://github.com/weaveworks/eksctl/releases/latest/download/eksctl_$(uname -s)_amd64.tar.gz" | tar xz -C /tmp
   sudo mv -v /tmp/eksctl /usr/local/bin
   ```
   Verify the installation:
   ```bash
   eksctl version
   ```

2. **Attempt to Create a Cluster**:
   Run the following command (replace `[YOUR-REGION]` with your AWS region, e.g., `us-west-2`):
   ```bash
   eksctl create cluster \
   --name nextwork-eks-cluster \
   --nodegroup-name nextwork-nodegroup \
   --node-type t2.micro \
   --nodes 3 \
   --nodes-min 1 \
   --nodes-max 3 \
   --version 1.31 \
   --region [YOUR-REGION]
   ```
   You'll encounter an error because your EC2 instance lacks the necessary permissions.

---

### Step 3: Create an IAM Role and Launch the EKS Cluster

1. **Create an IAM Role**:
   - Go to the **IAM Console**.
   - Create a new role with **AWS service** as the trusted entity and **EC2** as the use case.
   - Attach the **AdministratorAccess** policy.
   - Name the role: `nextwork-eks-instance-role`.

2. **Attach the IAM Role to Your EC2 Instance**:
   - Go to the **EC2 Console**.
   - Select your instance, click **Actions** > **Security** > **Modify IAM Role**.
   - Attach the `nextwork-eks-instance-role`.

3. **Create the EKS Cluster**:
   Run the `eksctl create cluster` command again. This time, it should succeed.

---

### Step 4: Monitor Cluster Creation with CloudFormation

1. Go to the **CloudFormation Console**.
2. Observe the stacks being created:
   - `eksctl-nextwork-eks-cluster-cluster` (core cluster resources).
   - `eksctl-nextwork-eks-cluster-nodegroup-nextwork-nodegroup` (node group resources).
3. Monitor the **Events** tab to track progress.

---

### Step 5: Access the EKS Cluster via AWS Management Console

1. Go to the **EKS Console**.
2. Select your cluster (`nextwork-eks-cluster`).
3. Grant yourself access:
   - Click **Create Access Entry**.
   - Select your IAM user and attach the `AmazonEKSClusterAdminPolicy`.
4. Verify that your nodes are listed under the **Compute** tab.

---

### Secret Mission: Test Cluster Resilience

1. **Terminate EC2 Instances**:
   - Go to the **EC2 Console**.
   - Terminate one or more nodes in your cluster.
2. **Observe Kubernetes Recovery**:
   - Kubernetes will automatically replace the terminated nodes.
   - Monitor the new nodes being created in the **EKS Console**.

---

## Cleanup

To avoid unnecessary charges, delete all resources:
1. **Delete the EKS Cluster**:
   - Go to the **CloudFormation Console**.
   - Delete the stacks: `eksctl-nextwork-eks-cluster-cluster` and `eksctl-nextwork-eks-cluster-nodegroup-nextwork-nodegroup`.
2. **Terminate the EC2 Instance**:
   - Go to the **EC2 Console**.
   - Terminate the `nextwork-eks-instance`.

---

## Conclusion

Congratulations! We have successfully:
- Launched and connected to an EC2 instance.
- Created a Kubernetes cluster using Amazon EKS.
- Monitored cluster creation with CloudFormation.
- Accessed and managed your cluster via the AWS Management Console.
- Tested the resilience of your Kubernetes cluster.

---

**Note**: Ensure all resources are deleted to avoid incurring charges. EKS is not AWS Free Tier eligible.
