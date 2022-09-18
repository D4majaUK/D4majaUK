---
layout: post
title: Azure Bicep - Get Started
categories:
  - bicep-post
---

# Azure Bicep development page

<img src="/Portfolio/images/bicep.png" width="100" height="100" />

## Get Started

Here's how to get up and running with Azure Bicep.

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

### Stretch the success

While this example is all good, it's not exactly what you would do in the real world. You would want to incorporate this into a CI/CD pipeline.
Azure DevOps would be the perfect companion for this. Create your project and inject the following yaml:
```
trigger:
- master

pool:
  vmImage: windows-latest

steps:
- task: AzureCLI@2
  displayName: Deploy Bicep Template
  inputs:
    azureSubscription: '<You will need to provide you Azure subscription here>'
    scriptType: 'ps'
    scriptLocation: 'inlineScript'
    inlineScript: >
      az deployment group create
      --mode complete
      --resource-group bicep 
      --template-file main.bicep
```
