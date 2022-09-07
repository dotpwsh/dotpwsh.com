---
title: 'Visualize Azure environment after deployment'
description: "In this article we're going to take a look at a Powershell module that can create a visualization of your Azure environment, and we're implementing it into Azure Pipelines to automatically run when a new deployment is done."
date: 2021-06-02T00:00:00
tags: ['Powershell', 'Azure Pipeline', 'Terraform']
draft: true
---

In this article we're going to take a look at a Powershell module that can create a visualization of your Azure environment, and we're implementing it into Azure Pipelines to automatically run when a new deployment is done. As a bonus we're uploading the images to Confluence. This way you will always have up-to-date overview of your environment.

A common setup today is to deploy infrastructure as code. There are various tools out there like Terraform, ARM and Pulumi to name a few. Having your infrastructure written as code is great for tracking changes, review processes and so on, but it's not always easy to visualize how different resources work together. For the most time I see infrastructure manually visualized in Visio or similar tools. The downside to this is that you must remmeber to add resources to your drawing as they are added. If you have multiple persons or teams deploying infrastructure it's very likely that it will be outdated quickly.

## Our project

First, we need something to visualize. Here's an overview of which resources I have in my subscription. If you have existing resources, thats fine. If you want the same setup as mine, to work along, here's a gist to get you setup.

```powershell
Connect-AzAccount

$resourceGroup = New-AzResourceGroup -Name "azure-visualizer-rg" -Location "westeurope"

$storageAccountSplat = @{
    ResourceGroupName = $resourceGroup.ResourceGroupName
    Location = $resourceGroup.Location

}

$storageAccountSplat = @{
  ResourceGroupName = $resourceGroup.ResourceGroupName
  Location = $resourceGroup.Location
}
New-AzStorageAccount @storageAccountSplat
```
