---
layout: post
title: Azure Bicep framework
categories:
  - bicep
---

# Azure Bicep development page

<img src="/Portfolio/images/bicep.png" width="100" height="100" />

## Get Started

Bicep is a domain-specific language (DSL) that uses declarative syntax to deploy Azure resources. In a Bicep file, you define the infrastructure you want to deploy to Azure, and then use that file throughout the development lifecycle to repeatedly deploy your infrastructure. Your resources are deployed in a consistent manner.

Bicep provides concise syntax, reliable type safety, and support for code reuse. Bicep offers a first-class authoring experience for your infrastructure-as-code solutions in Azure. [Learn more](https://docs.microsoft.com/en-us/azure/azure-resource-manager/bicep/overview)

## Pre-requisites

- Visual Studio Code
- Bicep extension for Visual Studio Code
  
  Search for *bicep* in the Extensions tab
  
  <img src="/Portfolio/images/bicep-extension.png" />
  
- Azure CLI (2.20.0 or later)
  
  Bicep comes with Azure CLI. Once installed, type **az bicep upgrade** to get the latest version.
  Typing **az bicep version** fives installed information.
  
## First Dip

Let's get right into it and create our first Azure Bicep template. Using Visual Code, create a file called **main.bicep**.

<img src="/Portfolio/images/AzureBicep-GetStarted.png" />

### Deploying first template

In order to rtun your first Azure Bicep template, we will need to first create a resource group in Azure, and then that will be used for the deployment.

```
az group create --name exampleRG --location uksouth
az deployment group create --resource-group exampleRG --template-file main.bicep --parameters storageName=uniquename
```

Check the resource group aand voila! you should see your resource group with the storage account.
