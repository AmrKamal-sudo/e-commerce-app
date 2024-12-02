# Microservices CI/CD

This repository contains a simple Flask app that serves user and product information via RESTful endpoints and we will Dockerize the application then provision AKS and configure a CI/CD pipeline using Azure DevOps to automate the build and deployment process. 

## Modifications

- **Requirements Update**:  
  The `requirements.txt` file has been modified to include a specific version of `Werkzeug==2.2.3`. This is because the latest version of `Werkzeug` that comes with Python does not support the Flask version used in this project, causing compatibility issues. The version `Werkzeug==2.2.3` ensures the Flask app works as expected.
- **run.py Update**:  
  the `run.py` file has been modified to include both host and port values.

## Getting Started

### Prerequisites

- Docker installed on your local machine.

### Installation

1. Clone the repository to your local machine:
   ```bash
   git clone https://github.com/AmrKamal-sudo/Microservices.git
   
   cd Microservices

Ensure the requirements.txt file includes the necessary dependencies, especially Werkzeug==2.2.3.

Build the Docker image:

    docker build -t flask-app .

Run the Flask app in a Docker container:

    docker run -p 5000:5000 flask-app

API Endpoints

Once the container is running, you can interact with the Flask app through the following endpoints:

Get all users

    curl http://localhost:5000/users

Get all products

    curl http://localhost:5000/products    
   
Get a specific user by ID
URL: http://localhost:5000/users/<id>
Example request for user ID 1:

    curl http://localhost:5000/users/1

Expected output:

{"id":1,"name":"John Doe"}

Get a specific product by ID
URL: http://localhost:5000/products/<id>
Example request for product ID 1:

    curl http://localhost:5000/products/1

Expected output:

{"id":1,"name":"Laptop"}

---

## We will outline two deployment methods for AKS: one using Terraform and the other by manually creating the AKS cluster via the Azure Portal.
---
#### For AKS creation with Terraform [Click Here](https://github.com/AmrKamal-sudo/Microservices/tree/main/Terraform-files)

#### In the following section I will guide you through the manual creation of an Azure Kubernetes Service (AKS) cluster using the Azure Portal, followed by the setup of a CI/CD pipeline for deploying our Python/Flask application."
---
## Prerequisites

Before starting, ensure you have the following:

- **Azure Subscription** with permissions to create resources.
- **Azure Container Registry** (ACR) to store Docker images.
- **Azure Kubernetes Service (AKS)** set up and running.
- **Azure DevOps Account** with permissions to create and manage pipelines.
- **GitHub Repository** with the Flask application code.

## Steps to Set Up

### 1. Create Azure ACR and AKS

#### Azure Container Registry (ACR)

1. Go to the **Azure Portal**.
2. Search for **Azure Container Registry** and create a new Registry to store your Docker images.

#### Azure Kubernetes Service (AKS)

1. In the **Azure Portal**, search for **Azure Kubernetes Services** and click **Create**.
2. Configure your Kubernetes cluster based on your needs (e.g., Node pools, OS, Networking, Monitoring, etc.).
3. Wait for the AKS deployment to finish.

### 2. Set Up Azure DevOps Project

1. Go to **Azure DevOps** and create a new project.
2. In **Azure DevOps**, go to **Pipelines** and click **Create Pipeline**.
3. Choose **GitHub** as your repository source and authenticate your GitHub account.
4. Select the repository containing the Flask app code.
5. In the **Pipeline Configuration**, choose **Deploy to Azure Kubernetes Service** as the template.

### 3. Configure the Pipeline

During the pipeline configuration, you will be asked to provide details about your Azure resources. Input the following:

- **Cluster Name**: The name of your AKS cluster.
- **New Namespace for the App**: A new namespace for your Flask app in AKS.
- **Container Registry**: The Azure Container Registry where your Docker image will be pushed.
- **Image Name**: The name of your Flask Docker image.
- **Service Port**: The port where the Flask app will run in AKS.

**Note**: You can either use Azure-hosted agents or configure a self-hosted Windows agent like what I used here in the guide ( `Dell-G15-SelfHosted`) for your pipeline.

### 4. CI/CD Pipeline Configuration

Here is the YAML configuration for the pipeline that automates the build and deployment of your Flask app `(Make sure you edit the resource names if you use the pipeline yaml file below to match your own resources)`.

```yaml
trigger:
- main

resources:
- repo: self

variables:
  # Container registry service connection established during pipeline creation
  dockerRegistryServiceConnection: 'fb190942-c36a-45af-a805-4c9760f0759a'
  imageRepository: 'flaskapp'
  containerRegistry: 'amrkamal.azurecr.io'
  dockerfilePath: '**/Dockerfile'
  tag: '$(Build.BuildId)'
  imagePullSecret: 'amrkamald586-auth'

  # Agent pool name for self-hosted Windows agent
  agentPool: 'Dell-G15-SelfHosted'

stages:
- stage: Build
  displayName: Build stage
  jobs:
  - job: Build
    displayName: Build
    pool:
      name: $(agentPool)
    steps:
    - task: Docker@2
      displayName: Build and push an image to container registry
      inputs:
        command: buildAndPush
        repository: $(imageRepository)
        dockerfile: $(dockerfilePath)
        containerRegistry: $(dockerRegistryServiceConnection)
        tags: |
          $(tag)

    - task: PublishPipelineArtifact@1
      displayName: Upload manifests as pipeline artifact
      inputs:
        targetPath: 'manifests'
        artifact: 'manifests'

- stage: Deploy
  displayName: Deploy stage
  dependsOn: Build

  jobs:
  - deployment: Deploy
    displayName: Deploy
    pool:
      name: $(agentPool)
    environment: 'AmrKamalsudoMicroservices.flask-app'
    strategy:
      runOnce:
        deploy:
          steps:
          - task: KubernetesManifest@0
            displayName: Create imagePullSecret
            inputs:
              action: createSecret
              secretName: $(imagePullSecret)
              dockerRegistryEndpoint: $(dockerRegistryServiceConnection)

          - task: KubernetesManifest@0
            displayName: Deploy to Kubernetes cluster
            inputs:
              action: deploy
              manifests: |
                $(Pipeline.Workspace)/manifests/deployment.yml
                $(Pipeline.Workspace)/manifests/service.yml
              imagePullSecrets: |
                $(imagePullSecret)
              containers: |
                $(containerRegistry)/$(imageRepository):$(tag)

```
---
5. Save and Run the Pipeline

Once the pipeline is configured, save the settings. This will push the pipeline configuration to your GitHub repository and trigger the build process.

The pipeline will trigger automatically when you push code to the main branch or when you manually run it from the Azure DevOps pipeline console. It will:

    1- Build the Docker image based on your Dockerfile.
    2- Push the Docker image to your Azure Container Registry.
    3- Generate artifacts such as deployment.yml and service.yml(Load Balancer) for Kubernetes.
    4- Deploy the Flask app to your Azure Kubernetes cluster.

6. Test the Deployed Application

Once the pipeline completes successfully, the Flask app will be deployed on your AKS cluster with a load balancer. You can access the app through the provided IP address.

here is my own deployed Load Balancer to test the deployed app:
Example commands to test the deployed app:

curl http://98.66.247.107:5000/users
Expected output: [{"id":1,"name":"John Doe"},{"id":2,"name":"Jane Doe"}]

curl http://98.66.247.107:5000/products
Expected output: [{"id":1,"name":"Laptop"},{"id":2,"name":"Smartphone"}]

