---
layout: post
title: Octopus Deploy Automation - Use Azure Tools
categories:
  - octopus-deploy-post
---

# Problem

> All Service Fabric deployments appear to now have a warning, due to an option with the GUI that is set as default.

| Azure Tools | Select whether to use the bundled Azure tools |
| --- | ----------- |
|   | (_) Use Azure Tools pre-installed |
|   | (X) Use Azure Tools bundled |

The warning is as follows:

```
Using the Azure tools bundled with Octopus Deploy is **not recommended.** 
Learn more about Azure Tools at https://g.octopushq.com/AzureTools.
```

# Solution

If you have a small amount of projects, you can quite simple go in and change each one back to **Use Azure tools pre-installed.**
However, if you have a bigger amount of projects, and don't want to painstakingly change each and everyone, there is Powershell.

After doing an initial Google search, I was able to find direction in how to change this programmatically.
###### You will have to have the Octopus Deploy API key and Octopus Deploy dll file in order to follow this method.
You will need references to Spaces Repository (A quick Google should be able to help you with this)
I created a project that deploys out a Service Fabric application. The deploy step contains the Azure Tools option.
From there you can do the following:

```
$projectInfo = $spacesRepo.Projects.FindByName("Bundled Demo")
write-host "Project Name:"$projectInfo.Name

# Get the process steps reference
$dp = $spacesRepo.DeployProcesses.Get($projectInfo.DeploymentProcessId)
$dp.Steps | % {
  if ($_.Name -eq "Deploy SF App") {
    <# Setting this to 'False' will mean using  Azure Tools pre-installed,
      the default that didn't appear to be set.#>
    $updValue = new-object Octopus.Client.Model.PropertyValueResource "False"
    $_.Actions.Properties["OctopusUseBundledTooling"] = $updValue
    $spacesRepo.DeploymentProcesses.Modify($dp)
  } 
}
```

It is as simple as that.
