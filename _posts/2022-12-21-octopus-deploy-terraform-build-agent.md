---
layout: post
title: Octopus Deploy Terraform Build Agent
categories:
  - octopus-deploy-post
---

# Problem

Octopus Deploy amongst many of it's capabilities, has the power to execute Terraform and build out infrastructure.
So, our task is to automate the creation of build agents and cut out the erroneous and inconsistent manual journey. 
Once the VM is created we will need to add some additional applications specific to build agents, via Chocolatey.

What I also wanted to add, was a little challenge, whereby the vnet and subnet are in a different resource group,
already exist and moreimportantly not under the power/control of Octopus Account used to create the build agent.
So this means that you hove to link to them, rather than create them.

# Solution

1. Create Storage account container for Terraform
2. Create an Octopus Runbook for the Build Agent implementation in Terraform
3. Add additional sofware via VM extension automation

### 1. Create Storage account for Terraform

This is really simple. Just go into the Azure portal and create a storage account, and within that storage account
you will need to create a container. The key part (you'll see what I did here) to this that this is the backend
state for Terraform. So it knows what it has controll over and won't try to create resources that it has already done.
You will need the access key from the storage account, in order for Octopus Deploy to write to that container.

### 2. Create an Octopus Runbook for the Build Agent implementation in Terraform

The core of this exercise is the Terraform template that Octopus Deploy uses to create the build agents.
The great thing about Octopus Deploy is that it allows you to enter the template directly in the custom project.
Meaning you can experiment until you get it right. Ideally you would put this under source control.
To get the same experience, I take it a stage further and create a package that acts the same it would under source control.
Also this helps with experiments, as you only have one template to update. In Terraform, you have a Plan & Apply step.

See below for the details on what is required. I also added a managed data disk, outside of this demo.

#### Terraform versions

So Octopus Deploy side of things goes through hefty testing to make sure Terraform works without fault (or at least minimum issues).
And so with that in mind, the version of Terraform will be some way back from the latest, even stable release.
But all is not lost, as you can actually force Octopus Deploy to use a later version, and it's quite simple.
Firstly, download the Terraform version you are comfortable to work with and locate it on the Octopus Deploy server, somewhere safe.
Then you will need to add a variable in the Runbook/Project you have set up for this task and as it's value, store the path and filename.

Variable name: **Octopus.Action.Terraform.CustomTerraformExecutable** 

Variable value: **C:\<folder location of file>\terraform.exe**

#### Challenges along the way

As I mentioned earlier, this required linking to a pre-existing vnet and subnet that was housed in a different resource group.
Also, the permissions will possibly be different and so each case will be different. The good thing about it,
is that Terraform/Octopus Deploy will be more than happy to let you know it failed, failed and failed again.
Here are some of the things you may expoerience and what is required to fix it.

For the vnet, you only need to have read permission [**Microsoft.Network/virtualNetworks/read**]

For the subnet, you only need to have join permission [**Microsoft.Network/virtualNetworks/subnets/join/action**]
> Note: this permission is added to the subnet resource, not the recipient. 

#### Terraform Template to create Build Agent infrastructure
```
terraform {
    required_version = ">= 1.3.6"
    backend "azurerm" {
        storage_account_name = "#{TF_StorageAccount}"
        container_name = "tf-state-buildagents"
        key = "terraform.tfstate"
        access_key = "#{TF_AccessKey}"
    }
}

provider "azurerm" {
    subscription_id = "#{TF_Sub}"
    client_id       = "#{TF_ID}"
    client_secret   = "#{TF_Scrt}"
    tenant_id       = "#{TF_Tenant}"
	features {}
}

data "azurerm_resource_group" "rg" {
  name     	= "#{TF_ResourceGroup}"
}

variable "windows_2019_sku" {
  type        = string
  description = "Windows Server 2019 SKU used to build VMs"
  default     = "2019-Datacenter"
}

variable "temp_username" {
  type        = string
  description = "Temporary username for this example"
  default     = "#{TF_TempUsername}"
}

variable "temp_password" {
  type        = string
  description = "Temporary password for this example"
  default     = "#{TF_TempPassword}"
}

variable "vm_naming" {
  type        = string
  description = "Name for the VM and additional components"
  default     = "<build agent name>"
}

resource "azurerm_network_interface" "nic" {
  name                = "${var.vm_naming}"
  location            = data.azurerm_resource_group.rg.location
  resource_group_name = data.azurerm_resource_group.rg.name

  ip_configuration {
    name                          = "${var.vm_naming}config"
    subnet_id                     = data.azurerm_subnet.subnet.id
    private_ip_address_allocation = "Dynamic"
  }

  tags = {
    Department   = "DevOps"
    Environment  = "Dev"
    Task         = "Network Interface"
  }
}


data "azurerm_virtual_network" "vnet" {
  name                = "<vnet name>"
  resource_group_name  = "<vnet resource group>"
}

data "azurerm_subnet" "subnet" {
  name                = "<subnet name>"
  resource_group_name  = "<subnet resource group>"
  virtual_network_name = data.azurerm_virtual_network.vnet.name
}

resource "azurerm_windows_virtual_machine" "vm" {
    name                  = "${var.vm_naming}"
    location              = data.azurerm_resource_group.rg.location
    resource_group_name   = data.azurerm_resource_group.rg.name
    network_interface_ids = ["${azurerm_network_interface.nic.id}"]
    admin_username     = var.temp_username
    admin_password     = var.temp_password
    size               = "standard_b2s"

    os_disk {
        name              = "${var.vm_naming}_DataDisk0"
        caching           = "ReadWrite"
	    storage_account_type = "Standard_LRS"
    }

    source_image_reference {
        publisher = "MicrosoftWindowsServer"
        offer     = "WindowsServer"
        sku       = "2016-Datacenter"
        version   = "latest"
    }

    tags = {
        Department   = "DevOps"
        Environment  = "Dev"
        Task         = "WindowsVM2019"
    }
}

resource "azurerm_virtual_machine_extension" "software" {
  name                 = "install-software"
  virtual_machine_id   = azurerm_windows_virtual_machine.vm.id
  publisher            = "Microsoft.Compute"
  type                 = "CustomScriptExtension"
  type_handler_version = "1.9"

  protected_settings = <<SETTINGS
  {
    "commandToExecute": "powershell -command \"[Net.ServicePointManager]::SecurityProtocol = [Net.SecurityProtocolType]::Tls12; iex ((New-Object System.Net.WebClient).DownloadString('https://chocolatey.org/install.ps1'))\""
  }
  SETTINGS

  tags = {
    Department   = "DevOps"
    Environment  = "Dev"
    Task         = "VMExtension"
  }
}


```
