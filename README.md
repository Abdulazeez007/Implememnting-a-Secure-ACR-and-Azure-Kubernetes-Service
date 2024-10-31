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
     ***az acr create --resource-group AuroraRG --name aurora$RANDOM$RANDOM --sku Basic***

![SOC]()
***Ensure to note down the registry Name***

## STEP 2: Create a Dockerfile, build a container and push it to Azure Container Registry.
- In the Bash session within the Cloud Shell pane, run the following to create a Dockerfile to create an Nginx-based image.

      ***echo FROM nginx > Dockerfile***

![SOC]()

### Next, run the following to build an image from the Dockerfile and push the image to the new ACR.

       ***ACRNAME=$(az acr list --resource-group AuroraRG --query '[].{Name:name}' --output tsv) az acr build --resource-group AuroraRG --image sample/nginx:v1 --registry $ACRNAME --file Dockerfile .***

- Might take a while to fully provision, but we can confirm that I’ve successfully pushed the dockerfile image to the ACR.
- Now, back to my Azure Portal,
- Resource group, in the container registry repository, we can see the image listed there, along with it’s SHA digest for Integrity.

![SOC]()

## STEP 3: Create an Azure Kubernetes Service cluster
- Back to Azure Portal, Search AKS or Azure Kubernetes Services,
- Configure and Create a new Kubernetes Cluster.
- In the Networking Tab, you’ll enable Azure CNI, ***this asigns IP Adresses to the pods and ensures that all the pods can be accessed individually.***
- review and create
![SOC]()

## Now we have created our AKS Cluster.
- On the Resource groups blade, in the listing of resource groups, note a new resource group named ***MC_AuroraRG_AuroraKubernetsCluster_centralus*** that holds components of the AKS Nodes.

![SOC]()

### Next, from the Azure Portal, navigate back to the CloudShell.
- In the Bash session within the Cloud Shell pane, run the following to connect to the Kubernetes cluster:
      ***az aks get-credentials --resource-group AuroraRG --name AuroraKubernetesCluster***
  
_ Then, In the Bash session within the Cloud Shell pane, run the following to list nodes of the Kubenetes cluster:
      ***kubectl get nodes***

![SOC]()

### We can confirm that the status of the cluster node is marked as Ready.

## STEP 4: Grant the AKS cluster permissions to access the ACR and manage its virtual network.
- within the CloudShell pane, configure the AKS cluster to use the Azure Container Registry created earlier.
- run the following command:

       ***ACRNAME=$(az acr list --resource-group AuroraRG --query '[].{Name:name}' --output tsv) az aks update -n AuroraKubernetsCluster -g AuroraRG --attach-acr $ACRNAME***

- This command grants the ‘acrpull’ role assignment to the ACR. ***the arcpull role assignment, allows ACR the permission to pull Images.***
- run the following to grant the AKS cluster the Contributor role to its virtual network.

       ***RG_AKS=AuroraRG***
       ***RG_VNET=MC_AuroraRG_AuroraKubernetsCluster_centralus***
       ***AKS_VNET_NAME=aks-vnet-73636518***  
       ***AKS_CLUSTER_NAME=AuroraKubernetsCluster***   
       ***AKS_VNET_ID=$(az network vnet show --name $AKS_VNET_NAME --resource-group $RG_VNET --query id -o tsv)***   
       ***AKS_MANAGED_ID=$(az aks show --name $AKS_CLUSTER_NAME --resource-group $RG_AKS --query identity.principalId -o tsv)***   
       ***az role assignment create --assignee $AKS_MANAGED_ID --role "Contributor" --scope $AKS_VNET_ID***
