---
title: "Monitoring Azure Automation Jobs"
date: 2023-06-18T13:59:36+02:00
draft: true
---

In this article we're going to explore how to monitor Azure Automation jobs and turn failed jobs into incidents. For this example we're going to integrate with ServiceNow.

## Throwing exceptions

By default, Azure Automation (and powershell in general) will only print the error message and continue the execution when encountering errors. This means that no excetion is thrown and the error is only available from the output.

There are numerous ways to handle errors in Powershell, but I like to be strict about it and use `$ErrorActionPreference = "Stop"` in my Azure Automation scripts. This means that when the script encouters an error it will throw an exception and stop execution. 

You can either let the original exception be thrown or use `try/catch` to maybe explain the problem in depth and provide your own error message. Either way, the exception will be displayed in the Azure Automation job log. The error message will be essential when we want to create our incident.

## Filters

In an Azure environment you probably have some Azure Automation Runbooks that are in production, and some for testing. Usually you dont need to report and create incidents for things that are in testing.

Therefor we want to setup a filter for which runbooks to include in our monitoring. This can be done using **tags** in Azure. You can tag an individual runbook. In this example we're going to filter which runbooks to monitor by looking for the tag "monitoring" and value "true". This makes it easy to include and exclude runbooks for monitoring.

## Azure Automation Account setup

For this monitoring setup we're going to use another Azure Automation Runbook, that is in its own Azure Automation Account.

For permissions to read the other runbooks, we're going to setup Managed Identity with the following roles:

- Automation Operator
- Automation Runbook Operator

We would also need the credentials for ServiceNow to be able to post an incident. Therefor it needs access to read secrets from Key Vault.

You can read more about how to setup Managed Identity for Azure Automation Accounts here: [Using a system-assigned managed identity for an Azure Automation account](https://learn.microsoft.com/en-us/azure/automation/enable-managed-identity-for-automation)

Now create a new runbook and give it a name (e.g "AzureAutomationAccountMonitoringToServiceNow"). We're going to choose "7.2 (preview)" as our runtime in this example, but you could also go with "stable" powershell 5.1 if thats a requirement.

## Scripting the runbook

### Initial setup

In our initial setup we're going to set `ErrorActionPreference` to "Stop", so that all our cmdlets will throw errors and stop executing. We're also going to store the name of our ServiceNow instance. This way we can easily switch between instances and its also clear which instance this script is targeting.

```powershell
$ErrorActionPreference = "Stop"
$ServiceNowInstanceName = "dev12345"
```

### Connecting to Azure using Managed Identity

Next, we need to authenticate to Azure in our Runbook to be able to read the other runbooks. Here's how we can do that using our managed identity.

```powershell
try {
    # ensures you do not inherit an AzContext in your runbook
    Disable-AzContextAutosave -Scope Process | Out-Null

    # connect to Azure with system-assigned managed identity
    $azureContext = (Connect-AzAccount -Identity).Context

    # set and store context
    $azureContext = Set-AzContext -Subscription $azureContext.Subscription -DefaultProfile $azureContext
} catch {
    Write-Error "failed to connect to az account: $_"
}
```


