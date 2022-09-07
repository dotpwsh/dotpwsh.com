---
title: 'Quicktip: Send output to clipboard'
description: "Ever found yourself wanting to copy the output from a command to your clipboard without having to select it first? Or maybe copy the contents of a file, like a config file? In this short article we're going to cover how to do that in Windows, MacOS and Linux with Powershell."
date: 2021-04-23T00:00:00
tags: ['Powershell', 'Windows', 'MacOS', 'Linux']
---

Ever found yourself wanting to copy the output from a command to your clipboard without having to select it first? Or maybe copy the contents of a file, like a config file? In this short article we're going to cover how to do that in Windows, MacOS and Linux with Powershell.

## The powershell way

Powershell 7 has a built-in cross platform cmdlet for both getting and setting your clipboard.

```powershell
PS> "Hello, World!" | Set-Clipboard
PS> Get-Clipboard
Hello, World!
```

or the shorter way

```powershell
PS> "Hello, World!" | scb
PS> gcb
Hello, World!
```

## Native commands

The built-in Powershell cmdlets are great, but just for fun, let's explore some other older commands native to it's platform.

### Windows

In Windows we have a command called `clip`. We can either pass a file or pipe output to this command and it will copy it to our clipboard.

#### Piping

Now let's say we want to fetch our public ip and copy it to clipboard we would do the following:

```powershell
irm ifconfig.co/ip | clip
```

If you `Ctrl+v` in your text editor of choice you should see the output of the command (in this example; your public ip).

#### Copy file contents

Let's say we want to share our Powershell profile settings, we could easily copy the contents:

```powershell
copy $profile
```

### MacOS

MacOS also comes with it's own command for copying output to clipboard, which is `pbcopy`.

Using the same example as in Windows, it will look like this:

```powershell
irm ifconfig.co/ip | pbcopy
```

Issuing `cmd + v` in a text editor should give you your public ip.

`pbcopy` does not support providing a file, like `clip` does, but we can easily achieve the same result by using `cat`and pipe the output to `pbcopy`.

```powershell
cat $profile | pbcopy
```

### Linux

Similar for Linux, we have the `xclip` command which looks a lot like the Windows version, but unlike Windows and MacOS, `xclip` does not come pre-installed in most Linux distributions. Luckily its available from all the major package managers.

**Ubuntu based systems**

```bash
sudo apt install xclip
```

**RHEL based systems**

```bash
sudo yum install xclip
```

**Arch based systems**

```bash
sudo pacman install xclip
```

#### Piping

Again, using the same example as for Windows and MacOS, this will copy your public ip to your clipboard in Linux.

```powershell
irm ifconfig.co/ip | xclip
```

#### Copy file contents

And like Windows version, `xclip` also supports providing a file to copy instead of having to use `cat`.

```powershell
xclip $profile
```

## Summary

As you see, this is nothing fancy, but can come in handy in several scenarios especially if you like to do most of your work in the terminal.
