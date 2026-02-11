---
date: "2025-11-11"
draft: false
title: "Create an OIDC Enabled Identity for Github Actions"
tags: ["github actions", "CI/CD"]
showHero: true
heroStyle: "background"
---

# Introduction

This PowerShell script was written to create an Entra service principal with a federated credential and role assignment at a scope selected during execution, for use with GitHub Actions performing deployments to Azure.

Although intended to use with GitHub Actions it could be re-purposed to set-up OIDC connections between Entra and other providers.

> [!TIP]+ Companion GitHub Repository
> The code detailed in this post can be found in this [repository](https://github.com/paul-mccormack/actions-entra-auth)
{icon="github"}

## Basic operation

You will be prompted to enter the required information to build the federated credential subject.

Ensure you have the appropriate permissions at the deployment scope you are deploying too. So if you need to PIM to get that do it before running the script. The script will exit if the operator does not have the permissions to create a role assignment at the desired scope. This is to prevent creating a service principal and federated credential without the role assignment.

During execution you will be prompted to login to Azure in PowerShell, if not already logged in. You then choose the scope for the role assignment, options are Management Group, Subscription and Resource Group.

If Resource Group is selected you will be prompted to select the subscription then enter the resource group name. Error handling is in place to catch typo's when entering the resource group name.

You will be prompted to choose the role for the role assignment from the following list of roles: ```PipelineAccess```, ```Owner``` and ```Contributor```.

> [!WARNING]
> Owner and Contributor should be used with caution, especially in Production.

The PipelineAccess role is a custom role following the Microsoft recomendation in the Azure DevOps documentation. See [here](https://learn.microsoft.com/en-us/devops/operate/governance-cicd#2-what-access-do-security-principals-need) for details of the role permissions.<br>
This role allows the creation of resources but denys ```"Microsoft.Authorization/*/Delete"``` actions. This includes the removal of management locks, deny assignments, policy assignments, role assignments and other highly privileged actions. The json file to create this role is included in the repository, named [CustomRole.json](https://github.com/paul-mccormack/actions-entra-auth/blob/main/CustomRole.json).

The values for ```AZURE_CLIENT_ID```, ```AZURE_TENANT_ID``` and ```AZURE_SUBSCRIPTION_ID``` are displayed after a successful run to be copied into your repository Actions Secrets section. These are very well known secret names commonly used with [azure/login@v2](https://github.com/marketplace/actions/azure-login) action.<br>
I have also included ```AZURE_OBJECT_ID``` for completeness as it's useful to have at times without resorting to digging around in the Entra portal. ```AZURE_SUBSCRIPTION_ID``` won't be output if the scope is a Management Group. Requiring the use of ```allow-no-subscriptions: true``` in the Azure Login action