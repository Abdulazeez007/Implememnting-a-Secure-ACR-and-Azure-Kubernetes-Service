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

![SOC](https://github.com/Abdulazeez007/Implementing-a-Secure-ACR-and-Azure-Kubernetes-Service/blob/main/AzureKubernetes/TOPOLOGY.jpg)

## STEP 1: Create an Azure Container Registry
- First, launch CloudShell from the Azure portal,
- In the Bash session within the Cloud Shell pane, run the following to create a new resource group.
     *** az group create --name AZ500LAB09 --location eastus***

![SOC](https://github.com/Abdulazeez007/Implementing-a-Secure-ACR-and-Azure-Kubernetes-Service/blob/main/AzureKubernetes/ACR%20Create.jpg)

### Next,create a new Azure Container Registry (ACR) instance.
     ***az acr create --resource-group AuroraRG --name aurora$RANDOM$RANDOM --sku Basic***

![SOC](https://github.com/Abdulazeez007/Implementing-a-Secure-ACR-and-Azure-Kubernetes-Service/blob/main/AzureKubernetes/ACR.jpg)

***Ensure to note down the registry Name***

## STEP 2: Create a Dockerfile, build a container and push it to Azure Container Registry.
- In the Bash session within the Cloud Shell pane, run the following to create a Dockerfile to create an Nginx-based image.

      ***echo FROM nginx > Dockerfile***

![SOC](https://github.com/Abdulazeez007/Implementing-a-Secure-ACR-and-Azure-Kubernetes-Service/blob/main/AzureKubernetes/Dockerfile.jpg)

### Next, run the following to build an image from the Dockerfile and push the image to the new ACR.

       ***ACRNAME=$(az acr list --resource-group AuroraRG --query '[].{Name:name}' --output tsv) az acr build --resource-group AuroraRG --image sample/nginx:v1 --registry $ACRNAME --file Dockerfile .***

- Might take a while to fully provision, but we can confirm that I’ve successfully pushed the dockerfile image to the ACR.
- Now, back to my Azure Portal,
- Resource group, in the container registry repository, we can see the image listed there, along with it’s SHA digest for Integrity.

![SOC](https://github.com/Abdulazeez007/Implementing-a-Secure-ACR-and-Azure-Kubernetes-Service/blob/main/AzureKubernetes/Dockerfile%20nginx.jpg)

## STEP 3: Create an Azure Kubernetes Service cluster
- Back to Azure Portal, Search AKS or Azure Kubernetes Services,
- Configure and Create a new Kubernetes Cluster.
- In the Networking Tab, you’ll enable Azure CNI, ***this asigns IP Adresses to the pods and ensures that all the pods can be accessed individually.***
- review and create
![SOC](https://github.com/Abdulazeez007/Implementing-a-Secure-ACR-and-Azure-Kubernetes-Service/blob/main/AzureKubernetes/CreateKub.jpg)

## Now we have created our AKS Cluster.
- On the Resource groups blade, in the listing of resource groups, note a new resource group named ***MC_AuroraRG_AuroraKubernetsCluster_centralus*** that holds components of the AKS Nodes.

![SOC](https://github.com/Abdulazeez007/Implementing-a-Secure-ACR-and-Azure-Kubernetes-Service/blob/main/AzureKubernetes/AKS%20Cluster.jpg)

### Next, from the Azure Portal, navigate back to the CloudShell.
- In the Bash session within the Cloud Shell pane, run the following to connect to the Kubernetes cluster:
      ***az aks get-credentials --resource-group AuroraRG --name AuroraKubernetsCluster***
  
_ Then, In the Bash session within the Cloud Shell pane, run the following to list nodes of the Kubenetes cluster:
      ***kubectl get nodes***

![SOC](https://github.com/Abdulazeez007/Implementing-a-Secure-ACR-and-Azure-Kubernetes-Service/blob/main/AzureKubernetes/GetNodes%20External.jpg)

### We can confirm that the status of the cluster node is marked as Ready.

## STEP 4: Grant the AKS cluster permissions to access the ACR and manage its virtual network.
- within the CloudShell pane, configure the AKS cluster to use the Azure Container Registry created earlier.
- run the following command:

       ***ACRNAME=$(az acr list --resource-group AuroraRG --query '[].{Name:name}' --output tsv) az aks update -n AuroraKubernetsCluster -g AuroraRG --attach-acr $ACRNAME***

- This command grants the ‘acrpull’ role assignment to the ACR. ***the arcpull role assignment, allows ACR the permission to pull Images.***
- Run the following to grant the AKS cluster the Contributor role to its virtual network.

       ***RG_AKS=AuroraRG***
       ***RG_VNET=MC_AuroraRG_AuroraKubernetsCluster_centralus***
       ***AKS_VNET_NAME=aks-vnet-73636518***  
       ***AKS_CLUSTER_NAME=AuroraKubernetsCluster***   
       ***AKS_VNET_ID=$(az network vnet show --name $AKS_VNET_NAME --resource-group $RG_VNET --query id -o tsv)***   
       ***AKS_MANAGED_ID=$(az aks show --name $AKS_CLUSTER_NAME --resource-group $RG_AKS --query identity.principalId -o tsv)***   
       ***az role assignment create --assignee $AKS_MANAGED_ID --role "Contributor" --scope $AKS_VNET_ID***
  
  - The contributor role grants read and write access to AKS Clusters.
  
![SOC](https://github.com/Abdulazeez007/Implementing-a-Secure-ACR-and-Azure-Kubernetes-Service/blob/main/AzureKubernetes/Contributor%20permission.jpg)

## STEP 5: Deploy an external service to AKS.
- Download the Manifest files, edit the YAML file, and apply your changes to the cluster.
- Download the Internal and External .yaml config file from Microsoft’s git: ***https://github.com/MicrosoftLearning/AZ500-AzureSecurityTechnologies/tree/master/Allfiles/Labs/09***
- In the Bash session within the Cloud Shell pane, click the Manage files icon, in the drop-down menu,
- Click Upload, in the Open dialog box, naviate to the location where you downloaded the lab files,
- Select nginxexternal.yaml click Open.
- Next, nginxinternal.yaml, and click Open.

![SOC](https://github.com/Abdulazeez007/Implementing-a-Secure-ACR-and-Azure-Kubernetes-Service/blob/main/AzureKubernetes/ExternalNginx.jpg)

**Now,** In the Bash session within the CloudShell pane, run the following to open the nginxexternal.yaml file, 

       ***code ./nginxexternal.yaml***
- In the editor pane, scroll down to line 24 and replace the <ACRUniquename> placeholder with the ACR name, which in my case is: ***aurora2608131553***
- click Save and then click Close editor.
- Within the CloudShell pane, run the following to apply the change to the cluster:

      ***kubectl apply -f nginxexternal.yaml***
![SOC]()
  
** This verifies that the deployment and the corresponding service have been created.

## STEP 6: Verify the you can access an external AKS-hosted service.
- Within the CloudShell pane, run the following to retrieve information about the nginxexternal service including name, type, IP addresses, and ports.

      ***kubectl get service nginxexternal***

![SOC]()

- Review the output and record the value in the External-IP column.

        ***48.214.188.216 ***
- Open a new browser tab and browse to the IP address.

**And here we have it live** 
![SOC]()

## STEP 7: Deploy an Internal Service to AKS
- Within the Cloud Shell pane, run the following to open the nginxintenal .yaml file, so you can edit its content:

       ***code ./nginxinternal.yaml***
  
- Scroll down to the line containing the reference to the container image and replace the <ACRUniquename> placeholder with the ACR name, like we did previously.
- In the Bash session within the CloudShell pane, run the following to apply the change to the cluster:

        ***kubectl apply -f nginxinternal.yaml***
-Next, within the CloudShell pane, run the following to retrieve information about the nginxinternal service including name, type, IP addresses, and ports.

       ***kubectl get service nginxinternal***
![SOC]()

- Review the output. The External-IP is, in this case, a private IP address.
- Take note of the External IP, in my case it’s:

         ***10.224.0.6***
- To access the internal service endpoint, you will connect interactively to one of the pods running in the cluster.

## STEP 8: Verify the you can access an internal AKS-hosted service
- Use one of the pods running on the AKS cluster to access the internal service.
- Within the CloudShell pane, run the following to list the pods in the default namespace on the AKS cluster:

        ***kubectl get pods***

  [SOC]()

- In the listing of the pods, copy the first entry in the NAME column.
- Within the CloudShell pane, run the following to connect interactively to the first pod.

      ***kubectl exec -it nginxexternal-6dcf4ff85d-l84kl -- /bin/bash***

![SOC]()

- Note how i added the name of the first pod to the command to connect to it.
- Now we’re in, I’m connected to the first pod within our bash pane.
- Within the CloudShell pane, run the following to verify that the nginx web site is available via the private IP address of the service.

      ***curl http://10.224.0.6***

![SOC]()

- We’ve successfully accessed the internal service.
- If we try to publicly access this service on my browser, we can observe that it won’t work, because it’s strictly an internal service that can only be accessed within the network.

![SOC]()  

## Conclusion and Key Takeaways

This project provided a hands-on experience with configuring and securing Azure Kubernetes Service (AKS) and Azure Container Registry (ACR) to efficiently manage containerized applications. The project objectives were effectively accomplished, showcasing the powerful integration of AKS and ACR in deploying scalable, secure applications on Azure's cloud platform. By completing this project, the workflow of creating, managing, and securing application containers on Azure was not only streamlined but also fortified with essential security practices, setting a foundation for best practices in cloud-native application development and deployment.

## Key Takeaways

1. **Robust Image Management with ACR**: Utilizing ACR enabled efficient storage, management, and versioning of container images, proving to be essential for consistent image availability and security within Azure. The secure image storage solution from ACR ensures container images are always available for AKS without risking unauthorized access.

2. **Seamless Integration of AKS with ACR**: Granting AKS permissions to access ACR highlighted the importance of controlled access in containerized environments. Configuring the `acrpull` role and `Contributor` role allowed for secure image retrieval and virtual network management, ensuring that only authorized services interact with sensitive resources.

3. **Network Isolation and Security**: Configuring both external and internal services within AKS emphasized the critical role of network policies. While external services were made accessible publicly, internal services remained securely isolated, showing how granular control can protect sensitive application components from external threats while allowing essential functionality to operate seamlessly within the network.

4. **Efficient Use of Azure CNI**: Enabling Azure CNI ensured that each pod received a unique IP address, reinforcing both accessibility and security. This setup allowed granular access control and precise traffic management, crucial for services requiring specific network configurations and a controlled security posture.

5. **Leveraging Kubernetes for Scalable Deployments**: The project illustrated AKS's capability to scale containerized applications seamlessly, showcasing its potential for managing large, distributed applications. AKS’s scalability and operational efficiency reaffirm its role as an optimal choice for enterprises looking to adopt containerization on a large scale.

6. **Practical Security and Operational Best Practices**: From managing credentials and roles to configuring private and public service access, the project emphasized a security-first approach to cloud infrastructure. The takeaway here is the importance of aligning both configuration and management practices to maintain a highly secure, operationally efficient cloud environment.

This project not only underscored essential Kubernetes and ACR skills but also strengthened understanding of Azure's role in deploying and securing scalable applications, setting the stage for advanced container orchestration and cloud-native architecture practices.

***This brings us to the conclusion of our lab, as we have successfully configured and secured both the Azure Container Registry (ACR) and Azure Kubernetes Service (AKS).***
