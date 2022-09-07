---
title: 'Azure: Getting started with Jekyll static website hosting'
description: "In this multipart series we're going through the steps of hosting a jekyll static website in Azure and automate as much of the process as possible. We will cover everything from installing Jekyll to setting up custom domain (with HTTPS) and analytics."
date: 2019-11-29T00:00:00
tags: ['Azure', 'Powershell']
draft: true
---

In this multipart series we're going through the steps of hosting a jekyll static website in Azure and automate as much of the process as possible. We will cover everything from installing Jekyll to setting up custom domain (with HTTPS) and analytics.

Parts in this series:

1. [Azure Static Website: Getting started with Jekyll static website hosting](/2019/11/29/azure-getting-started-with-jekyll-static-website-hosting/)
2. Azure Static Website: Setting up a 'static website'-enabled Storage Account for Jekyll
3. Azure Static Website: Setting up 'Build' and 'Release' in Azure Pipelines for Jekyll
4. Azure Static Website: Using Azure CDN for custom domain with HTTPS
5. Azure Static Website: Website analytics through Azure Application Insights

## Install Jekyll

To get started you must install Jekyll to build a new Jekyll site. The official Jekyll site has already covered this so I'll just link to it. It covers installing Jekyll on Ubuntu Linux, which also works in Windows Subsystem for Linux (WSL).

[Jekyll on Ubuntu](https://jekyllrb.com/docs/installation/ubuntu/)

Once Jekyll is installed we can generate our new site with following command, where `my-azure-site` will be the name of the folder created in current working directory.

```bash
jekyll new my-azure-site
```

Now lets `cd` into our new site and serve it locally to take a look at it.

```bash
cd my-azure-site
bundle exec jekyll serve
```

Your site is now typically available at `http://127.0.0.1:4000/`. Open the URL in your browser and see what Jekyll has created for us.

> For further instructions on how to customize your site, please visit the official documentation: [Jekyll Step by Step Tutorial](https://jekyllrb.com/docs/step-by-step/01-setup/)

Now that we know our site is working we can continue with setting up Azure DevOps.

## Setting up Azure DevOps

For those who dont know what Azure DevOps is you can read about it here [Azure DevOps Services](https://azure.microsoft.com/nb-no/services/devops/).

If you dont already have a Azure DevOps account, you can sign up for free [here](https://go.microsoft.com/fwlink/?LinkId=2014881&campaign=acom~azure~devops~services~main~hero&githubsi=true&clcid=0x414&WebUserId=0298EDA0252C66F90538E381212C604E).

### Introduction

We're going to use two of the services offered by Azure DevOps;

- Azure Repos
- Azure Pipelines

**Azure Repos** is Microsoft's answer to Github (even though they now own both). This is where we're going to store our code.
**Azure Pipelines** is a service to setup Build and Release Pipelines (CI/CD). We're going to use this to trigger the build and upload to Azure Storage when we push commits to the master branch.

### Create new project

If you haven't already, log into your Azure DevOps account and click the "New Project" button in the top right corner. Fill in the following information. Project name is the only thing that is 'required'. The rest we're going to leave at default.

| **Project name** | Give your project a name, i.e `my-jekyll-blog` |
| **Description** | Optionally. A brief description of your project |
| **Visibility** | Public or Private. Default private. For this project we choose `private` |
| **Advanced/Version Controll** | Default Git. We're going to use git |
| **Advanced/Work item process** | Default Agile. Not relevant for our project |

Click on "Create" when you have filled in everything and your project will be created and you will be redirected to the projects dashboard.

In meny to the left you can see the Azure DevOps services that is available to your project. As mentioned, we're only going to use Pipelines and Repos.

### Commit Jekyll site to Azure Repos

Now that our project is created we can start pushing our Jekyll code to the repository.

#### Authenticate with SSH keys

Before we can push code we must authenticate ourself, and the easiest way to do this on linux is by SSH keys. If you dont know what SSH keys are, or dont have a SSH key pair, you can read about it here: [Github: Generating a new SSH key and adding it to the ssh-agent](https://help.github.com/en/github/authenticating-to-github/generating-a-new-ssh-key-and-adding-it-to-the-ssh-agent)

When you have generated your SSH key (or if you already have one), you can run the following command to print out your public key, which we will paste into Azure DevOps.

```bash
cat ~/.ssh/id_rsa.pub
```

1. Copy whats outputted and go to Azure DevOps.
2. In the top right corner, there's an icon wih a person and a cog. Click on it and then **SSH Public Key**.
3. Click **New Key** in the top right corner.
4. Give it a name, ie `my-awesome-laptop`, and paste the output of the `cat` command.
5. Click **Add**.

Git can now automatically authenticate when pushing or pulling from Azure Repos you have access to from your laptop.

> **NOTE:** If you have multiple laptops, you must do the same on them.

#### Initialize git and first commit

To push commits to our repo, we must first initialize it locally and set a remote endpoint to push to.

Remember the `my-jekyll-blog` site we created locally earlier? Go back to that folder and initialize git.

```bash
git init
```

A local git repository is now created for that entire folder.

Now lets add and commit everything in our site (you can run `git status` to see which files will be added).

```bash
git add .
git commit -m "Initial commit"
```

#### Push code to Azure Repos

Go back to Azure DevOps and click on your project. Then click on **Repos**. You should have an empty repository and a messages telling you how to set it up.

Copy what under "or push an existing repository from command line" and run it locally to add our remote endpoint.

```bash
git remote add origin https://example@dev.azure.com/example/my-jekyll-blog/_git/my-jekyll-blog
git push -u origin --all
```

git will now push everything to our Azure Repos `my-jekyll-blog`. When it's done you can refresh the Azure Repos site and see that your files are now displayed.

## Summary

We've installed Jekyll and made sure it runs. Then we created a new Azure DevOps project to hold our code and added our ssh keys for easy authentication when pushing or pulling. Finally we initialized git locally, commited and pushed our code to Azure Repos.

Next time we're going to setup our Azure Storage account that is going to hold the output of Jekyll (a.k.a our static website).
