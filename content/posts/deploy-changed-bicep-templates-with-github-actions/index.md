---
title: "Deploy Changed Bicep Templates with GitHub Actions"
date: 2026-01-23
draft: false
tags: ["bicep", "github actions", "azure", "CI/CD"]
showHero: true
heroStyle: "background"
---

[![Deploy](https://github.com/paul-mccormack/deploy-changed-bicep-templates/actions/workflows/deploy.yml/badge.svg)](https://github.com/paul-mccormack/deploy-changed-bicep-templates/actions/workflows/deploy.yml)

# Introduction 
This project is to develop a method for checking if Bicep templates in the repo have changed on the latest commit.  If they have add them to the deployment.  If they haven't skip them.  The ultimate goal is to reduce the run time for a deployment on a project that contains multiple sets of templates and parameter files that are unlikley to be changed and redeployed together after the initial deployment.

## The problem I'm trying to solve

I wanted to do this to improve the pipeline config for a repository that is used to manage Azure Policy.  That project consists of four seperate sets of Bicep templates each with a corresponding bicepparam file. Currently the pipeline is setup to trigger when a push is made to the main branch and will perform the checks and deployment for all the templates within the repo regardless of whether they were changed in the last commit or not.  This is not a real problem for the resulting policy deployment stacks due to the idempotent natue of Bicep but it does mean each deployment takes an excessive amount of time.  A more desirable configuration would be to have some method of detecting which of the Bicep templates or parameter files have changed and only perform the actions required for that template.

To test the pipeline operation in this repo I am deploying an Azure Storage Account and a Vnet as test resources that don't incur cost and can be easily redeployed to change configuration.

## The solution - git diff to the rescue!

The solution I have developed uses the [git diff](https://git-scm.com/docs/git-diff) command to look for the changed files in the last commit, then in the pipeline jobs I have used a PowerShell [switch](https://learn.microsoft.com/en-us/powershell/module/microsoft.powershell.core/about/about_switch?view=powershell-5.1) statement to perform an action if the name of the bicep or bicepparram file is found within the output from the git diff command.

The code below shows this:

```Powershell
Write-Output "Checking for changed Bicep and parameter files in the last commit..."
$changed = git diff HEAD HEAD~1 --name-only | Select-String -Pattern '.bicep'
if ($null -ne $changed) {
  Write-Output "The following Bicep or parameter files have changed in the last commit:"
  $changed
  foreach ($item in $changed) {
    switch ($item) {
      'stg.bicep' { New-AzResourceGroupDeployment -Location ${{ env.location }} -ResourceGroupName ${{ env.rgName }} -TemplateFile ./stg.bicep -TemplateParameterFile ./stg.bicepparam -Verbose }
      'stg.bicepparam' {  New-AzResourceGroupDeployment -Location ${{ env.location }} -ResourceGroupName ${{ env.rgName }} -TemplateFile ./stg.bicep -TemplateParameterFile ./stg.bicepparam -Verbose }
      'vnet.bicep' { New-AzResourceGroupDeployment -Location ${{ env.location }} -ResourceGroupName ${{ env.rgName }} -TemplateFile ./vnet.bicep -TemplateParameterFile ./vnet.bicepparam -Verbose }
      'vnet.bicepparam' { New-AzResourceGroupDeployment -Location ${{ env.location }} -ResourceGroupName ${{ env.rgName }} -TemplateFile ./vnet.bicep -TemplateParameterFile ./vnet.bicepparam -Verbose }
    }
  }
} else {
  Write-Output "No Bicep or parameter files have been changed in the last commit."
}
```

If the storage account related files appear in the list of changed files from git diff the command to deploy the storage account template will be run.  If the vnet related file names appear the command to deploy the vnet will be run.  If a file from both the storage account and the vnet files was changed in the last commit both templates will be deployed. 

To make this work your actions runner needs to have the repo checked out.  This is very easy to do with the checkout action but I was having problems which suggested the git history wasn't being included in that checkout.  I found the answer was to include the ```fetch-depth``` option in the checkout step.  You can control how much history is checked out with this statement also.  Including ```fetch-depth: 0``` will bring the entire history.  This might take a bit of time for a very large repo.  In theory we only need the most recent commit and the one previous to that.  The statement ```fetch-depth 1``` should give you that.

In the pipeline config file it looks like this:

```yml
- name: Checkout repository
  uses: actions/checkout@v5
  with:
    fetch-depth: 0  # Fetch all history for all branches and tags
```

The pipeline config file is [deploy.yml](https://github.com/paul-mccormack/deploy-changed-bicep-templates/blob/main/.github/workflows/deploy.yml)

## Testing the deployment

For the first test I want to get both resources deployed.  This proves the pipeline will be able to detect changes to multiple templates or parameter files and act accordingly.  Creating a change in both parameter files and pushing that to the main branch results in the pipeline running, from the logs we can see the results are exactly as we expect.  Both the storage account and the vnet have been picked up as changed and are being deployed.

```
Checking for changed Bicep and parameter files in the last commit...
The following Bicep or parameter files have changed in the last commit:

stg.bicepparam
vnet.bicepparam
VERBOSE: Using Bicep v0.37.4
VERBOSE: Calling Bicep with arguments: build-params "/home/runner/work/deploy-changed-bicep-templates/deploy-changed-bicep-templates/stg.bicepparam" --stdout
VERBOSE: Performing the operation "Creating Deployment" on target "rg-uks-sandbox-pmc".
VERBOSE: 16:19:11 - Template is valid.
VERBOSE: 16:19:42 - Resource Microsoft.Storage/storageAccounts 'stguumu27r4tuhry' provisioning status is succeeded
.
VERBOSE: Using Bicep v0.37.4
VERBOSE: Calling Bicep with arguments: build-params "/home/runner/work/deploy-changed-bicep-templates/deploy-changed-bicep-templates/vnet.bicepparam" --stdout
VERBOSE: Performing the operation "Creating Deployment" on target "rg-uks-sandbox-pmc".
VERBOSE: 16:20:01 - Template is valid.
VERBOSE: 16:20:15 - Resource Microsoft.Network/virtualNetworks 'vnet-pmc-test' provisioning status is succeeded
VERBOSE: 16:20:32 - Resource Microsoft.Network/virtualNetworks/subnets 'vnet-pmc-test/subnet1' provisioning status is succeeded
```

For the second test I need to make a change to one of the resources and ensure the deployment only picks that template to run.  To do that I will disable public network access on the storage account and commit the change to the repo.  Again from the logs this is working as expected.  Only the storage account parameter files was picked up and deployed.

```
Checking for changed Bicep and parameter files in the last commit...
The following Bicep or parameter files have changed in the last commit:
stg.bicepparam
VERBOSE: Using Bicep v0.37.4
VERBOSE: Calling Bicep with arguments: build "/home/runner/work/deploy-changed-bicep-templates/deploy-changed-bicep-templates/stg.bicep" --stdout
VERBOSE: Performing the operation "Creating Deployment" on target "rg-uks-sandbox-pmc".
VERBOSE: 16:49:57 - Template is valid.
VERBOSE: 16:50:03 - Resource Microsoft.Storage/storageAccounts 'stguumu27r4tuhry' provisioning status is succeeded
```

The final test is to make a commit that doesn't include the bicep templates or parameter files.  Again from the logs it performed exactly as required.  No Bicep deployments were made.

```
Checking for changed Bicep and parameter files in the last commit...
No Bicep or parameter files have been changed in the last commit.
Script execution Complete
```


## Conclusion

After implementing these changes to the pipeline configuration in my Azure Policy repository, run times dropped from an average 9 minutes down to 3 minutes when a change was made to a single bicep template or parameter file.  Resulting in a much more efficient config and potentially preventing the need to increase resources and cost by requiring the purchase of concurrent runners for the environment.

🦾