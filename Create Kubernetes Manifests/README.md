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

## Steps Overview

1. **Create an ECR Repository**: Store the Docker image in Amazon ECR for easy access by Kubernetes.
2. **Push Docker Image to ECR**: Build the Docker image and push it to ECR.
3. **Create Kubernetes Manifests**: Define how Kubernetes should deploy and expose the Flask app using deployment and service manifests.
4. **Deploy to EKS**: Deploy the app to a running EKS cluster using `kubectl`.
5. **Clean Up Resources**: Delete all resources to avoid incurring charges.

---

## Creating ECR Repository

![73shots_so (1)](https://github.com/user-attachments/assets/cf72b8cd-2f0a-4af1-9c84-a27c23618a27)

1. **Create ECR Repository**  
Run the following AWS CLI command to create an ECR repository for storing your Docker images:

    ```bash
    aws ecr create-repository \
      --repository-name nextwork-flask-backend \
      --image-scanning-configuration scanOnPush=true
    ```

2. **Verify Repository**  
After running the above command, verify the repository is created by navigating to the ECR console. You should see the `nextwork-flask-backend` repository.

---

## Pushing Docker Image to ECR

1. **Tag the Docker Image**  
Make sure your Docker image is tagged with the ECR repository URI:

    ```bash
    docker tag YOUR-IMAGE:latest YOUR-ECR-URI/nextwork-flask-backend:latest
    ```

2. **Push the Image**  
Push the tagged image to ECR:

    ```bash
    docker push YOUR-ECR-URI/nextwork-flask-backend:latest
    ```

3. **Verify Image in ECR**  
After the push completes, you can verify the image in the ECR console.

---

## Creating Kubernetes Manifests

1. **Create the Deployment Manifest**  
Create a new directory called `manifests` and navigate into it:

    ```bash
    mkdir manifests
    cd manifests
    ```

    Create the `flask-deployment.yaml` file:

    ```yaml
    apiVersion: apps/v1
    kind: Deployment
    metadata:
      name: nextwork-flask-backend
      namespace: default
    spec:
      replicas: 3
      selector:
        matchLabels:
          app: nextwork-flask-backend
      template:
        metadata:
          labels:
            app: nextwork-flask-backend
        spec:
          containers:
            - name: nextwork-flask-backend
              image: YOUR-ECR-URI/nextwork-flask-backend:latest
              ports:
                - containerPort: 8080
    ```

2. **Create the Service Manifest**  
Create the `flask-service.yaml` file to expose your app:

    ```yaml
    apiVersion: v1
    kind: Service
    metadata:
      name: nextwork-flask-backend
    spec:
      selector:
        app: nextwork-flask-backend
      type: NodePort
      ports:
        - port: 8080
          targetPort: 8080
          protocol: TCP
    ```

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
