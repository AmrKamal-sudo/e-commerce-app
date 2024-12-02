# Get Started

Before you begin, make sure you have the following:

## 1. Azure Account
You need an Azure account to deploy the resources. If you don't have one, create one now to follow along.

## 2. Install Azure CLI
Install the latest version of the Azure CLI on your machine. After installation, run the following command to log in to your Azure account:

    az login

## 3. Install Terraform



## 4. Install kubectl


## 5. Configure and run Terraform 


Run terraform init to initialize the Terraform deployment. This command downloads the Azure provider required to manage your Azure resources.

    terraform init -upgrade
---
    


Run terraform plan to create an execution plan for AKS resources.

    terraform plan -out main.tfplan
---



Run terraform apply to apply the execution plan to your Azure cloud infrastructure.

    terraform apply main.tfplan
---
    

The following commands for Windows machines using Command Prompt (CMD)

Get the Kubernetes configuration from Terraform output and store it in a file that kubectl can read using the following command.

    terraform output kube_config > azurek8s

Note: If you see << EOT at the beginning and EOT at the end, remove these characters from the file. Otherwise, you may receive the following error message: error: error loading config file "./azurek8s": yaml: line 2: mapping values are not allowed in this context

---

Set an environment variable so kubectl can pick up the correct config using the following command.

    set KUBECONFIG=.\azurek8s

---

Verify the health of the cluster using the kubectl get nodes command.

    kubectl get nodes
