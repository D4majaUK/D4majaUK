---
layout: post
title: Octopus Deploy Automation - Deploy Azure Bicep Templates
categories:
  - octopus-deploy-post
---

# Problem

There is a need to deploy Azure Bicep templates using Octopus Deploy.

# Solution

This is surpisingly starightforward to achieve. The best way to trial this out, is by using Octopus Deploy Runbooks.
The deployment agents had Azure CLI installed, but didn't have Azure Bicep installed. 
Quite simply, this was a case of adding a line in the script to install it, and Bob's your mother's brother.
You can use inline code, and for a quick experiment, this is possibly the preferred option.
Let's not do that :)

What I want to show you is how you would achieve this in a simple but useful example.
And for that I will be using the Octopus Deploy built-in functionality of packages.

Below is an Azurer Bicep example of adding a storage account to Azure. You name it something meaningful, 
and no weirdness. Just make sure the extension is .bicep extension.

```
param location string

resource storageAcc 'Microsoft.Storage/storageAccounts@2022-09-01' = {
  name: '**<appropriate name>**'
  location: location
  kind: 'StorageV2'
  sku: {
    name: 'Standard_RAGRS'
  }
}
```

Ok, so in order to be able to use this file, we need to package it up. I know it's a single file at the miniute, 
but this will help you going forward when you have multiple files to deal with.
Octopus also has a wonderful CLI tool, called **octo**. Being an automation-focused individual, this helps.
So the manual way is to zip up the files, name it as *<name.major.minor.fix>.zip*. Then you upload that package to Octopus Deploy.
The CLI allows you to automate mate, creating something like this:- (I'm using Powershell)
All you have to do is pass in the version number as a parameter. I also store my files in a subfolder called *Templates*.
It will create the zip file and then upload it, overwriting the existing version if required. 
You can either leave this in or make sure that you are giving a new version each time.

```
cls
write-host "Octo Package for Build Agent creation"
write-host " "
if ($args) {
	$buildNum = $args[0]
	octo pack --id="<package name you want to call>" --format="zip" --version=$buildNum --basePath="Templates" --overwrite

	$pkgName = "<package name defined above>." + $args[0] + ".zip"
	write-host " "
	write-host "Build package name: " $pkgName
	write-host " "
	write-host "Uploading package: " $pkgName " to Octopus Deploy"
	octo push --package="$pkgName" --replace-existing
} else {
	write-host "you need to give a version number (eg. 1.0.1)"
}
write-host " "

```

Now we have the package, you are then able to include it in the Runbook.

<img src="/Portfolio/images/ODaddPackage.jpg" />

Make sure you pick **RUN AN AZURE SCRIPT** step for this, and it will require an Azure subscription to link to.

You will need to use the *Inline Source Code* option and put in some basic lines to get it working.

```
az bicep install

$filePath = $OctopusParameters["Octopus.Action.Package[tcbicepbuild].ExtractedPath"]
cd $filePath


az deployment group create --resource-group *<destination resource group name>* --template-file *<your bicep filename>*.bicep --parameters location='*<geolocation>*'
```

This will go through the following steps:
* Install Azure Bicep (assuming as default it has not been installed)
* Redirect to where the package that you included has been extracted to
* And then execute against the location and resource group that you provide

Run the Runbook and when it has finished, you can check your Azure resource group to see that it has deployed as expected.

That is pretty much everything you need to know to get started. And then you just build on that.
