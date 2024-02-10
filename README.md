# Automate Azure cloud infrastructure setup using Ansible and Azure DevOps pipeline | How to integrate Ansible with Azure DevOps | Integrating Ansible with Azure DevOps Pipelines |

Ansible is an open-source configuration management tool that automates cloud provisioning, configuration management, and application deployments. Using Ansible you can provision virtual machines, containers, network, and complete cloud infrastructures.

## Integrate Ansible with Azure Cloud
Integrating Ansible with Microsoft Azure allows you to automate and manage your Azure infrastructure using Ansible playbooks and modules. Ansible provides a collection of Azure-specific modules that enable you to provision and configure resources in Azure.

# To configure Azure credentials, you need the following information:
Your Azure subscription ID and tenant ID
The service principal application ID and secret

Pre-requisites:
Azure account subscription, click here if you don't have one.
Azure CLI needs to be installed.
Service principal to create any resources in Azure cloud using Azure cloud shell or Azure CLI
# step 1 Login to Azure
'az login'
Enter Microsoft credentials

# step 2 Create Azure Service Principal
Run the following commands to create an Azure Service Principal:

az ad sp create-for-rbac --name <service-principal-name> \ 
--role Contributor \ 
--scopes /subscriptions/<subscription_id>

Save the above output in a file as you will not be able retrieve later.

# step 3 Create an Ansible playbook
Create a simple playbook to create resource group in Azure. Make sure you modify the name of the resource group and location below.

---

- hosts: localhost

  connection: local

  tasks:

    - name: Creating resource group

      azure_rm_resourcegroup:

        name: "my-rg12"

        location: "eastus"


# step 4 Create Azure YAML build pipeline:
Login to Azure Devops --> https://dev.azure.com

Select project dashboard.

Go to Pipelines -> New pipeline --> Click on Azure Repos Git or any SCM where you have playbooks stored. Select repo, click on Starter pipeline.
Add below four pipeline variables with value received from service principal creation.

AZURE_SUBSCRIPTION_ID
AZURE_CLIENT_ID
AZURE_SECRET
AZURE_TENANT

# step 5 Add below tasks:
Install Ansible on build agent
Install Ansible rm module on build agent
Execute Ansible playbook for creating resource group in Azure cloud.

trigger:
- main
pr: none  # Disable PR triggers, can be adjusted as needed
pool:
  vmImage: 'ubuntu-latest'
steps:
- script: |
    #Install Ansible
    pip3 install "ansible==2.9.17"
  displayName: 'Install Ansible'
  
- script: |
    #Install Ansible rm module
    pip3 install ansible[azure]
  displayName: 'Install Ansible rm module'
  
- script: |
    #Run Ansible playbook to create resource group
    ansible-playbook create-rg.yml
  displayName: 'Run Ansible Playbook'
  env:
    AZURE_SUBSCRIPTION_ID: $(AZURE_SUBSCRIPTION_ID)
    AZURE_CLIENT_ID: $(AZURE_CLIENT_ID)
    AZURE_SECRET: $(AZURE_SECRET) 
    AZURE_TENANT: $(AZURE_TENANT) 

Save the pipeline and run it.

Now Login to Azure cloud to see if the resource group have been created.