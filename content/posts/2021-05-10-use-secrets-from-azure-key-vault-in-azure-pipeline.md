---
title: "Use secrets from Azure Key Vault in Azure Pipelines"
date: 2021-05-10T00:00:00
tags: ['Powershell', 'Bash', 'Azure Pipeline', 'Azure Key Vault', 'Terraform']
draft: true
---

The chance of you needing to use some kind of secrets in your Azure Pipeline is big. At some point your pipeline wants to talk to a third party service and there will be tokens or some kind of credential involved. In this article we're taking a look at how to fetch and use secrets from Azure Key Vault in your pipeline.

Azure Key Vault is one of the more popular secrets management tools on the market (HashiCorp Vault, AWS Secret Manager, Thycotic Secret Server are some of the alternatives). It's accessible from everywhere and has great API and SDK support.

We're going to use a simple Terraform deployment as example. Our goal is to fetch the credentials that Terraform needs to connect to our tenant and remote statefile.

## Project setup

For the sake of this article I'm assuming that

* you already have a [Azure Pipeline Service Connection](https://docs.microsoft.com/en-us/azure/devops/pipelines/library/service-endpoints?view=azure-devops&tabs=yaml) setup in your pipeline that has access to a Key Vault
* you know how a basic azure pipeline yaml file works

This is our skeleton for `build.yaml`:

```yaml
trigger:
  branches:
    include:
      - master
  paths:
    include:
      - /*

pr: none

variables:
  - name: TF_IN_AUTOMATION
    value: true
  - name: System.Debug
    value: true

stages:
  - stage: Build
    jobs:
      - job: Build

        pool:
          vmImage: 'ubuntu-latest'

        steps:
          - checkout: self
            fetchDepth: 1

          - task: AzureCLI@1
            inputs:
              azureSubscription: 'AzKeyVault-ServiceConnection'
              scriptLocation: 'scriptPath'
              scriptPath: './scripts/fetch-credentials.sh'
              failOnStandardError: true
            displayName: 'Fetch credentials'
```

As you might have noticed our first task is the 'Fetch credentials' which uses an `AzureCLI@1` task. This task basically gives us bash shell with azure cli installed and connected to our service connection. That way, we can use `az keyvault secret` module to fetch our secrets.

So, let's start building our `fetch-credentials.sh` script.

```bash
set -euo pipefail

BACKEND_ACCESS_KEY=$(az keyvault secret show --name "terraform_backend_access_key" --vault-name "terraform_credentials" --query value -o tsv)
CLIENT_ID=$(az keyvault secret show --name "terraform_client_id" --vault-name "terraform_credentials" --query value -o tsv)
CLIENT_SECRET=$(az keyvault secret show --name "terraform_client_secret" --vault-name "terraform_credentials" --query value -o tsv)
```

We start of by fetching our credentials from Azure KeyVault. Notice that we do not have to issue a login command, because the task we're using has already authenticated us using the service connection. So as long as the service connection has access to the Key Vault, there should be no problem. Note that the service connection also must have access to read the secrets, it's not enough that it has access to the Key Vault itself. Make sure that you've setup access policies for the service connection.

Now, our problem is that we want to use these credentials in other scripts and tasks in our job, without having to fetch them each time. The solution is to use something Microsoft is calling [logging commands](https://docs.microsoft.com/en-us/azure/devops/pipelines/scripts/logging-commands?view=azure-devops&tabs=bash#logging-command-format) and more specific the [SetVariable](https://docs.microsoft.com/en-us/azure/devops/pipelines/scripts/logging-commands?view=azure-devops&tabs=bash#setvariable-initialize-or-modify-the-value-of-a-variable) command.

The gist of it is that we can set variables by printing to stdout with a specific syntax, and Azure Pipeline will set these variables as environment variables throughout our job. The syntax is quite simple:

```
##vso[task.setvariable variable=<name>;]<value>
```

You can also specify some properties:

* issecret
* isoutput

`issecret` will treat the variable as a secret variable and prevent it from being outputted in the pipeline. This will also prevent it from being used in other scripts without passing it explicitly as a environment variable first (we'll go through it soon).

`isoutput` will set the variable as an output variable for the current job, which means that we can pick it up and use it in other jobs. So if you want your variable to be available cross jobs, you must specify this.

Let's get back to our example and set the credentials we fetched as variables in our pipeline.

```bash
echo "##vso[task.setvariable variable=TF_BACKEND_ACCESS_KEY;issecret=true]${BACKEND_ACCESS_KEY}"
echo "##vso[task.setvariable variable=TF_CLIENT_ID;issecret=true]${CLIENT_ID}"
echo "##vso[task.setvariable variable=TF_CLIENT_SECRET;issecret=true]${CLIENT_SECRET}"
```

Since these are credentials and should be kept as secret as possible, we're setting the `issecret=true`.

As with every Terraform deployments we must start with a `terraform init` so let's add that to our pipeline.

```yaml
# ... previous code ...

- task: ms-devlabs.custom-terraform-tasks.custom-terraform-installer-task.TerraformInstaller@0
  displayName: 'Install Terraform 0.14.6'
  inputs:
    terraformVersion: 0.14.6

- task: Bash@3
  inputs:
    filePath: './scripts/terraform-init.sh'
    arguments: '$(Build.SourcesDirectory)'
    workingDirectory: '$(Build.SourcesDirectory)'
    failOnStderr: true
  env:
    ARM_ACCESS_KEY: $(TF_BACKEND_ACCESS_KEY)
    ARM_CLIENT_ID: $(CUST_CLIENT_ID)
    ARM_CLIENT_SECRET: $(CUST_CLIENT_SECRET)
  displayName: 'Terraform Init'
```

First we use a built in Azure Pipeline task to install a specific version of Terraform (`v0.14.6` in this case).
