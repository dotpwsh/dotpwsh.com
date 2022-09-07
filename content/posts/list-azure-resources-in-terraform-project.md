---
title: 'List Azure resources in Terraform project'
description: "Some Terraform projects can grow really big and I can be hard to keep track of which Azure resources are in use. Recently I found myself wanting to know which Azure resources our Terraform project use so that I could document them internally. It could also be helpful in case there has been some major changes to some resources in the azurerm Terraform provider that you want to check."
date: 2021-06-03T00:00:00
tags: ['Powershell', 'Terraform']
---

Some Terraform projects can grow really big and I can be hard to keep track of which Azure resources are in use. Recently I found myself wanting to know which Azure resources our Terraform project use so that I could document them internally. It could also be helpful in case there has been some major changes to some resources in the azurerm Terraform provider that you want to check.

This, not so little, one-liner will list all `azurerm_` and `azuread_` resources uniquely in your entire Terraform project if you run it from your root module.

<script src="https://gist.github.com/madsaune/943f2124375625700f00dbe2f150c28b.js"></script>
