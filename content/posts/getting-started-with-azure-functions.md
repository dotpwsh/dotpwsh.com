---
title: 'Getting started with Azure Functions'
description: "Azure Functions is one of Microsofts serverless services that you can setup in Azure. Being serverless means that you dont have to worry about the infrastructure and environment behind it and you will only pay for the capacity that you actually use when the function is running. In this articles we'll look at how to setup an Azure Functions App"
date: 2020-02-19T00:00:00
tags: ['Azure Functions', 'Azure CLI']
published: true
---

Azure Functions is one of Microsofts serverless services that you can setup in Azure. Being serverless means that you dont have to worry about the infrastructure and environment behind it and you will only pay for the capacity that you actually use when the function is running. Traditionally you would have a server that runs 24/7 and consume capacity. Very simplified, serverless will spin up and down as requests comes in. This means that you only have to focus on the code it should run.

## Prerequisites

1. An Azure Subscription
2. Azure CLI ([Install the Azure CLI](https://docs.microsoft.com/nb-no/cli/azure/install-azure-cli?view=azure-cli-latest))
3. Node.js and npm

You can register for a free Azure subscription here: [https://azure.microsoft.com/en-us/free/](https://azure.microsoft.com/en-us/free/)

## Introduction

In this article I will walk you through the following subjects.

1. Create an Azure Function
2. Install Azure Functions Core Tools and create a new local function
3. Publish function to Azure Function

## Create an Azure Function

### Connecting to Azure

The first task is to login to Azure through our CLI.

```bash
az login
```

This will try to open your browser and direct you to a login page, if not you will be instructed to go to [https://microsoft.com/devicelogin](https://microsoft.com/devicelogin) and type in a token that is also provided.

```bash
dotpwsh@demo:~$ az login
WARNING: To sign in, use a web browser to open the page https://microsoft.com/devicelogin and enter the code <TOKEN> to authenticate.
```

Now that we're logged in we can start working with Azure.

### Preparation

To setup an Azure Function we're going to need the following resources from Azure.

- Resource Group
- Storage Account
- Azure Function

Each of them is going to need, among other things, a name and location. To make it easier for us let's prep some variables.

```bash
LOCATION=westeurope
RESOURCE_NAME=af-test-rg
STORAGE_NAME=$(head /dev/urandom | tr -dc A-Za-z0-9 | head -c 12)-af-test-sa #E.g SdE4npmV81ok-af-test-sa
FUNCTION_NAME=$(head /dev/urandom | tr -dc A-Za-z0-9 | head -c 12)-test-af #E.g 48aDF9HYfiWY-test-af
```

> It's a good practice to append a shortcode to the end of Azure resource name's that says something about what it is. E.g 'sa' for storage account, 'rg' for resource group, 'af' for Azure Functions and so on.

Every resource in Azure needs to live somewhere, so we have to specify a location. In this example I'm going to use `westeurope` for all the resources because it's the closest for me, but you should choose something close to you.

To list available locations you can run these two commands.

```bash
az account list-locations -o table
az functionapp list-consumption-locations -o table
```

We also create a name variable for each of our resources. The resource group name only needs to be unique within your subscription, but both storage account and function app names must be globally unique. Therefor we're append a random string of characters infront of it. The storage account name can only be between 3 and 24 characters long.

How you choose to generate the random string is up to you.

### Create a Resource Group

Briefly, every service in Azure must belong to a resource group. You can have one or more services within a resource group. For the most part you want to use resource groups as a logical collection of services related to a project or deployment. E.g a website might need an App Service, Storage and some Analytics. Services related to that specific website should reside in the same resource group, so when the day come that you want to decomission it, you can just delete the whole resource group. It also gives you a better overview of which services are related.

Let's start by creating our resource group.

```bash
az group create --name $RESOURCE_NAME --location $LOCATION
```

The `--name` parameter is what we would like to call our resource group. As mentioned this must be unique within your subscription. The `--location` parameter is where we want our resource group to be stored. Since we have already decided on the name and location, we can use the variables we setup earlier.

### Create our Storage Account

Next, we need a storage account where our files (code and modules) will be stored. Azure Functions does not include storage on its own.

```bash
az storage account create \
--name $STORAGE_NAME \
--location $LOCATION \
--resource-group $RESOURCE_NAME \
--sku Standard_LRS
```

The only thing here that we havent defined beforehand is the type of storage we want. The `--sku` paramter is how we define which type we want. Here we use the simplest one which is 'Standard_LRS'.

### Create our Function app

Lastly, we're going to create our Azure Function app.

```bash
az functionapp create \
--name $FUNCTION_NAME \
--resource-group $RESOURCE_NAME \
--consumption-plan-location $LOCATION \
--storage-account $STORAGE_NAME \
--runtime node
```

Here we create the Azure Function app and link it to the resources we created earlier. One important parameter is the `--runtime` parameter which define what kind of runtime you want. This can be one of the following: dotnet, java, node, powershell or python.

In this example we want our function to run nodejs (Javascript) code, so we specify `node`. For now, the default Azure Function version is 2, so in our example it will create a nodejs 10 runtime environment for us. You could specify a `--functions-version 3` and then `--runtime-version 12` to get the latest nodejs runtime environment. For simplicity we're going to stick with the defaults.

> You can read more about the available options here: [az functionapp create](https://docs.microsoft.com/en-us/cli/azure/functionapp?view=azure-cli-latest#az-functionapp-create)

When this command has finished, our Function app is up and running, but we havent deployed any code yet, so let's do that.

### Bonus: Azure Function appsettings

A Function app can have **Application settings** which are exposed as environmental variables in your code. Let's say you have one or more functions (within your Function app) that does something and then sends a notification to a Microsoft Teams channel. You can then expose the Teams Webhook Uri as environmental variables instead of hardcoding them in each function. This makes it very easy to send the notification to another channel.

```bash
az functionapp config appsettings set \
--name $FUNCTION_NAME \
--resource-group $RESOURCE_NAME \
--settings TEAMS_URL=<YOUR_TEAMS_WEBHOOK_URL>
```

The command needs the name of the Function app and the resource group that the Function app is in, then you can specify the settings with `<VARIABLE_NAME>=<VALUE>` format. If you want to specify multiple settings you can separate them with spaces. If the setting already exists, it will replace it's value. **NOTICE: That there's no space between the 'variable_name' and the 'value'**.

## Install Azure Functions Core Tools and create a new local function

Now that we have set everything up in Azure, we're ready to start developing our functions, but first we need to install **Azure Functions Core Tools** which will help us to generate a function template and publish it to Azure when we're done.

### Installing Azure Functions Core Tools

Azure Functions Core Tools is a npm package, so we need to install it through npm.

```bash
npm install -g azure-functions-core-tools
```

This will install the **azure-functions-core-tools** package globally, which means that we dont have to install for every project. By default this will install the package for Function app version 2. If your Function app version is 3, you must specify the version, like so:

```bash
npm install -g azure-functions-core-tools@3
```

### Create our Function app project

When the azure-functions-core-tools package is done installing, we can generate a new project like this. This will create a _azurefn-test-project_ folder in your current directory with a set of standard files.

```bash
func init azurefn-test-project
```

You will be prompted for a runtime. In this example choose `node` and `javascript`. When done, `cd` into your project folder.

```bash
cd azurefn-test-project
```

Now our project is setup, but we haven't yet created any functions.

### Create our function inside the project

```bash
func new --name HelloFromOutside
```

This command will create our function. You will be prompted for which template you want to choose. In this example choose "Http Trigger", which will give us a template for a function that runs when its requested by HTTP. If you e.g want to do something scheduled you can choose "Timer Trigger".

> You can create multiple functions by invoking `func new` and give it a name. Every function you create inside this project will be published to our Function app in Azure.

Our project structure should now look like this.

```plaintext
dotpwsh@demo:~$ tree .
.
├ HelloFromOutside
│   ├ function.json
│   └ index.js
├ host.json
├ local.settings.json
└ package.json

1 directory, 5 files
```

Our function itself resides in the _HelloFromOutside_ folder and the code that runs is in the _index.js_ file.

The function generated from the `func new` command will look for a name parameter either in the queryString or the body of the request and send back "Hello \<name\>". We're not going to build out a fancy function, since it's not the scope of this article, but you can do all sorts of things here. This will do for now.

### Testing our function

To test your functions locally you can run the `func start` command to spin up a local server which will give you an URI that you can hit through HTTP (e.g your browser, curl, postman or whatever).

## Publish function to Azure Function

When you're done writing and testing your function it's time to publish it to Azure Function so that it's available from the outside. This is a oneline command that will do everything for you, just make sure you are logged in with `az login`.

```bash
func azure functionapp publish $FUNCTION_NAME
```

This command might take some time to finish. Essentially this command will send your code to Azure, build it, and deploy it to your Function app.

> You can read more about the `func azure functionapp publish` options here [Publish to Azure](https://docs.microsoft.com/en-us/azure/azure-functions/functions-run-local?tabs=windows#publish)

When it's finished publishing you will get, among other things, a "Invoke url" back. Copy this, and verify that your function is working. **NOTE: The code in the URI will differ from function to function**.

```bash
dotpwsh@demo:~$ curl https://48aDF9HYfiWY-test-af.azurewebsites.net/api/HelloFromOutside?code=HmPfMj4ImBd7VwrL31IO47OZlLataj2A6LMw6mkQA4sbJSLQiEDtNm==&name=dotpwsh

Hello dotpwsh
```

## Wrapping up

In this article we've gone over how to create an Azure Function in Azure through the Azure CLI. We have created a Function app locally, tested it and published it to Azure. All of this is fairly straight forward and can easily be automated. The next step would be to create a repository in Azure DevOps and then automatically build and publish it to your Function app on commit to master branch or some other trigger. That will be for another article.

Have fun creating your Azure Function apps. The sky is the limit for what this can use this for!
