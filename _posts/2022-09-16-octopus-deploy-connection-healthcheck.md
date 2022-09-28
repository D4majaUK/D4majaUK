---
layout: post
title: Octopus Deploy Self-Heal Health Check
categories:
  - octopus-deploy-post
---

# Problem

Octopus Deploy performs health checks to every target virtual machine (vm) that it is connected to. For this example, the vm is in Azure. 
For whatever reason (overnight shutdown and follow-on startup, random restart, etc), sometimes the connection is not re-established.
The manual resolve to this is to restart the each vm in Azure, and then retry the health check connectivity in Octopus Deploy.
What would be good, is if this could be accomplished through automation, as it is a clearly defined repeatable set of steps.

# Solution

1. Check Octopus Deploy Health-check status
2. Check Azure status
3. Run Octopus Deploy Connectivity Ping

#### Check Octopus Deploy Health-check status

Using the Octopus Deploy Client API, we can query the status of the connectivity to the target vms.
Firstly, get the vm machine ids and health state, then store them, separating out the vms with an unhealthy state. 

```
$machineStatus = @()
$machineUnhealthy = @()
$machineList = $spaceRepository.Machines.GetAll()
$machineList | where {$_.EnvironmentIds -contains "<Environment you are interested in>" -and $_.Name -like "<you can enter specific machine names>*"} | % {
      $psObject = [PSCustomObject]@{
          Name = $_.Name
          Status = $_.Status
          HealthStatus = $_.HealthStatus
      }
	$machineStatus += $psObject
	if (($_.HealthStatus -ne "Healthy" -or $_.Status -ne "Online")) {$machineUnhealthy += $psObject}
}
```

Next we move onto Azure, using Azure Powershell to restart the vms that have failed connectivity checks.
As you can see below that it is quite a simple procedure, I have just added some extra reporting to show the state of the vm.
You may have to do some additional work if your vms are located in different resource groups.
```
foreach ($vmName in $vmArray) {
  $result = Get-AzureRmVM -ResourceGroupName "<resource group where the vm is located>" -Name $vmName -Status
  $result.Statuses | % {
    write-host "VM Name:       "$vmName
    write-host "Code:          "$_.Code
    write-host "DisplayStatus: "$_.DisplayStatus
    write-host "Time:          "$_.Time
    write-host " "
  }
  Restart-AzureRmVM -ResourceGroupName "<resource group where the vm is located>" -Name $vmName
  
  # As an alternative, you can stop and then start the VM
  # Stop-AzureRmVM -ResourceGroupName "<resource group where the vm is located>" -Name $vmName
  # Start-Sleep -Seconds 5
  # Start-AzureRmVM -ResourceGroupName "<resource group where the vm is located>" -Name $vmName
}
```

Finally, we can then run the Octopus Deploy Health-check to re-test the connection between Octopus Deploy and the vm in Azure.
Providing you have an array of vms from the intial health-check status, you can flow through the example below.
The Octopus Deploy Health-check re-test requires that you provide machine IDs. So first we build a list of IDs to re-test, against the vm list.
Then set up the parameters to feed into the API call. Execute the API call, wait 30 seconds, and then get the status of the health-check re-test. 
Again, we have added displaying the return from the calls, which can help with troubleshooting, if required.
```
$Macs = $spaceRepo.Machines.GetAll()
$MachineIDs = @()
foreach ($vm in $vmArray) {
	$Macs | where-object {$_.Name -eq $vm} | % {$MachineIDs += $_.Id}
}

$Description = "Health check started from 'AUTOMATED HEAL' Powershell script"
$TimeOutAfterMinutes = 5
$MachineTimeoutAfterMinutes = 5
$EnvironmentID = "<octopus deploy environment id that you are focused on>"

$healthCheckId = $spaceRepo.Tasks.ExecuteHealthCheck($Description,$TimeOutAfterMinutes,$MachineTimeoutAfterMinutes,$EnvironmentID,$MachineIDs)
$healthCheckId | fl *
$healthCheckId.Arguments | fl *
Start-Sleep -Seconds 30
$state = $spaceRepo.Tasks.Get($healthCheckId.Id)
$state | fl *
```
