# Implememnting-a-Secure-ACR-and-Azure-Kubernetes-Service
This project focuses on advanced configuration and security hardening of Azure Container Registry (ACR) and Azure Kubernetes Service (AKS) to ensure a robust containerized environment. Key objectives include implementing secure access controls, enabling image scanning and vulnerability assessments in ACR, and configuring network policies, role-based access controls, and workload identity management within AKS. Additionally, the project applies best practices for securing communication between ACR and AKS, with a focus on minimizing attack surfaces, enforcing compliance policies, and enhancing overall threat detection and response capabilities in container orchestration.

## KEY OBJECTIVES

1. 🛠️ **Create an Azure Container Registry (ACR)**
2. 🐳 **Create a Dockerfile, build a container, and push it to Azure Container Registry**
3. ☁️ **Create an Azure Kubernetes Service (AKS) cluster**
4. 🔑 **Grant the AKS cluster permissions to access the ACR**
5. 🌐 **Deploy an external service to AKS**
6. ✅ **Verify access to the external AKS-hosted service**
7. 🏢 **Deploy an internal service to AKS**
8. ✅ **Verify access to the internal AKS-hosted service**

## NETWORK TOPOLOGY


## STEP 1: Create an Azure Container Registry
- First, launch CloudShell from the Azure portal,
- In the Bash session within the Cloud Shell pane, run the following to create a new resource group.
     *** az group create --name AZ500LAB09 --location eastus***

![SOC]()

### Next,create a new Azure Container Registry (ACR) instance.
     ***az acr create --resource-group AuroraRG --name Aurora$RANDOM$RANDOM --sku Basic***

![SOC]()
***Ensure to note down the registry Name***

## STEP 2: Create a Dockerfile, build a container and push it to Azure Container Registry.
- In the Bash session within the Cloud Shell pane, run the following to create a Dockerfile to create an Nginx-based image.

      ***echo FROM nginx > Dockerfile***
  
### Next, run the following to build an image from the Dockerfile and push the image to the new ACR.

       ***ACRNAME=$(az acr list --resource-group AZ500LAB09 --query '[].{Name:name}' --output tsv)
 az acr build --resource-group AZ500LAB09 --image sample/nginx:v1 --registry $ACRNAME --file Dockerfile .***


