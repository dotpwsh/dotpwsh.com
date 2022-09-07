---
title: 'Improving Your Powershell Profile (Windows)'
description: "For years I've been a fan of the linux bash, with easy support for ssh-keys, colorized directory listings and git info the prompt. But at the same time, I really love Powershell. I have finally found some usefull Powershell modules that has made me switch completly to Powershell in the terminal."
date: 2020-06-18T00:00:00
tags: ['Powershell', 'Windows']
published: true
---

For years I've been a fan of the linux bash, with easy support for ssh-keys, colorized directory listings and git info the prompt. But at the same time, I really love Powershell. I have finally found some usefull Powershell modules that has made me switch completly to Powershell in the terminal.

## Colorized Directory Listings

The first module I'm going to introduce is the **Get-ChildItemColor** module by Joon Ro ([github.com/joonro/Get-ChildItemColor](https://github.com/joonro/Get-ChildItemColor)).

This module will override the `Out-Default` cmdlet and give you colorized directory listings when using `Get-ChildItem` or `ls`.

You can easilly install it from the [Powershell Gallery](https://www.powershellgallery.com/packages/Get-ChildItemColor).

```powershell
Install-Module -Name Get-ChildItemColor -Scope CurrentUser -AllowClobber
```

**NOTE:** The `-AllowClobber` flag is neccessary for it to override the `Out-Default` cmdlet.

Now you can just add `Import-Module -Name Get-ChildItemColor` to your Powershell profile.

## Git Information In Your Prompt

The second module we're going to add is the `posh-git` module by Keith Dahlby ([github.com/dahlbyk/posh-git](https://github.com/dahlbyk/posh-git)). This will override your default prompt and add git information when in a folder with git initialized. **NOTE:** This will not override your custom prompt, if you have defined one in your Powershell profile.

This module is also available from the [Powershell Gallery](https://www.powershellgallery.com/packages/posh-git). Currently, v1.0 is in beta, and is neccessary if you want support for Powershell Core 6.0 and up. Version v0.x only supports Windows Powershell.

```powershell
Install-Module -Name posh-git -Scope CurrentUser -AllowPrerelease -Force
```

To be able to install the v1.0-beta we must include the `-AllowPrerelease` flag.

Next, just add it to your Powershell profile `Import-Module -Name posh-git`.

## Using SSH Keys With Remote Git Repositories

The last module is `posh-sshell` which is a helper module for your SSH client and used to be a part of the `posh-git` module. This has now been separated into it's own module by the same creator Keith Dahlby ([github.com/dahlbyk/posh-sshell](https://github.com/dahlbyk/posh-sshell)).

As with the others, this is available from the [Powershell Gallery](https://www.powershellgallery.com/packages/posh-sshell).

```powershell
Install-Module -Name posh-sshell -Scope CurrentUser
```

There is one cmdlet in particular that we're interrested in, which is the `Start-SshAgent` cmdlet. This will start your SSH agent wether you're using the Windows-native OpenSSH client, OpenSSH client that ships with Git for Windows or putty's Pageant client.

If you are using the Windows-native OpenSSH client, make sure that the service is not `disabled`.

```powershell
Get-Service -Name ssh-agent | Select-Object Status, Name, StartType
```

If `StartType` says `disabled` you can run the following command to enable it or else `Start-SshAgent` will fail.

```powershell
Get-Service -Name ssh-agent | Set-Service -StartType Manual
```

Next, add the following to your profile.

```powershell
Import-Module -Name posh-sshell
Start-SshAgent -Quiet
```

When the ssh-agent is started it will look for ssh-keys in your `$env:USERPROFILE\.ssh` folder. If you add SSH keys after the ssh-agent has started you can either restart it with

```powershell
Stop-SshAgent
Start-SshAgent
```

or add it to the ssh-agent with

```powershell
# Adds $env:USERPROFILE\.ssh\id_rsa to the SSH agent.
Add-SshKey

# OR
# Adds $env:USERPROFILE\.ssh\mykey to the SSH agent.
Add-SshKey ~\.ssh\mykey
```

## Conclusion

Now, atleast in my opinion you have a more similar workflow in Powershell that you would have in linux. It's really cool to see how far Powershell (and Windows) has come in the field of developer workflow.
