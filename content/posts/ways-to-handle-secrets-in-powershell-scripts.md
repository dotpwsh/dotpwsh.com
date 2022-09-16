---
title: "Ways to Handle Secrets in Powershell Scripts"
description: "In this article I'm briefly looking at some options for securely handling secrets in Powershell scripts, so that we can avoid hard coded credentials and strengthen our security"
date: 2022-09-16T20:29:26+02:00
tags: ["Powershell", "Azure", "Security"]
image: "ways-to-handle-secrets-in-powershell-scripts"
---

In this article I'm briefly looking at some options for securely handling secrets in Powershell scripts, so that we can avoid hard coded credentials and strengthen our security.

## Service Principal with certificate

One of the most common approches when using a service principal is to setup a password (secret), so that you authenticate using a `client_id` and `client_secret`. But if these are stored in a script they can easily be exploited. So, the better option is to use self-signed certificates. This way you must have the certificate installed on the machine that run the script. This is often used for scripts that talks to Microsoft/Azure services API's, like Microsoft Graph or Azure Resource Manager (ARM).

Documentation: [Use Azure PowerShell to create a service principal with a certificate](https://docs.microsoft.com/en-us/azure/active-directory/develop/howto-authenticate-service-principal-powershell)

## Managed Service Identity (MSI)

Managed Service Identites is an automatically managed identity in Azure Active Directory for applications that can be used to authenticate to resources that supports Azure AD authentication. This way you dont have to manage any credentials yourself (like secrets, tokens or certificates). A MSI is only granted for a specific instance of an application like an Azure Functions App or Azure Runbook, and you assign permissions this application. This is the recommended way, but not all resources and applications supports MSI. And it only works in Azure.

Documentation: [Managed Identites for Azure Resources](https://docs.microsoft.com/en-us/azure/active-directory/managed-identities-azure-resources/overview)

## Azure Key Vault

So far we have mostly covered Azure services, but what if you need to fetch credentials for a third party service, like ServiceNow, Jira or something else. You can't use a service principal with certificate or a MSI for these services because they are outside the Azure ecosystem. One way is to store the credentials in Azure Key Vault, and then use a service principal with certificate or a MSI to access it from your script.

Documentation: [Azure Key Vault documentation](https://docs.microsoft.com/en-us/azure/key-vault/)

## CI/CD Variables

Most CI/CD solutions supports storing secrets for use in pipelines. If you have some static secrets/tokens that your script needs to access, you should store them in a secret variable within your CI/CD tool instead of hard coding them in your script or pipeline definition. Most tools will make these secrets available through environment variables when your pipeline is run.

- In Github Actions they are called "Action secrets".
- In Azure Pipelines they are called "Variables" with the option to make them secret or not.

Documentation

- [Encrypted secrets](https://docs.github.com/en/actions/security-guides/encrypted-secrets)
- [Azure Pipelines - Define variables](https://docs.microsoft.com/en-us/azure/devops/pipelines/process/variables?view=azure-devops&tabs=yaml%2Cbatch)

## SecretManagement and SecretStore

SecretManagement is a Powershell module that provides a common set of cmdlets to interact with secrets vaults (SecretStore is one of them). By using these common cmdlets in your scripts, you dont have to worry about what kind of secrets vaults your colleagues are using.

There are many vault extensions that work with SecretManagement like KeePass, LastPass, Azure Key Vault and some others. You can search for the tag "secretmanagement" in the Powershell gallery to find more extensions ([PowerShell Gallery: Packages matching Tags:"SecretManagement"](https://www.powershellgallery.com/packages?q=Tags%3A%22SecretManagement%22))

Works great for shared local scripts.

Documentation: [Overview of the SecretManagement and SecretStore modules](https://docs.microsoft.com/en-us/powershell/utility-modules/secretmanagement/overview?view=ps-modules)

## Group Managed Service Accounts (gMSA)

I dont have experience with this option, but I have read about it. Group Managed Service Accounts sounds like MSI, but for Windows machines. As they explain it in the documentation:

"When a gMSA is used as service principals, the Windows operating system manages the password for the account instead of relying on the administrator to manage the password."

You can read all about it here: [Microsoft Docs: Group Managed Service Accounts Overview](https://docs.microsoft.com/en-us/windows-server/security/group-managed-service-accounts/group-managed-service-accounts-overview)
