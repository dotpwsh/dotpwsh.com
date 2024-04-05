---
title: "Why you should consider learning Powershell this year"
date: 2024-04-05T00:00:00+02:00
tags: ['Powershell']
---

For many, Powershell is associated with Windows. But did you know that it works just as good on MacOS and several Linux distributions? Ever since Powershell 6 (named Powershell Core) it has had support for running on multiple platforms, and it keeps getting better at each new version. Powershell is now at version 7.4.1 at the time of writing, and Powershell 7.5 is around the corner.

I've been using Powershell as my main shell on MacOS since about two years ago. That's one of the great things about Powershell; its both a shell and a programming language. One major benefit of this is that its really easy to test out small pieces of code, or concepts before adding them to your program. And you can use your skills in multiple situations.

I think I can say with confidence that most people look to Python as the default programming language for automation. It has been written tons of articles and books on how to automate tasks with Python. In this article I want to show you that Powershell is a viable option to Python when it comes to automation. Let's take a look at what makes Powershell so great.

## What makes Powershell great?

### It's cross-platform and works on all the major platforms and architectures

Currently, Powershell 7.4+ is supported on the following platforms and architectures:

- Windows (x86, x64, arm32, arm64)
- Debian (amd64)
- Alpine (x64)
- RHEL (x64)
- Ubuntu (x64, arm32)
- Raspberry Pi OS
- MacOS (x64, arm64)

### Easy to install

Powershell comes pre-installed on Windows and Azure Cloud Shell. It's also mostly a oneline command away from installation.

```powershell
# Windows
winget install --id Microsoft.Powershell --source winget

# Debian / Ubuntu
sudo dpkg -i powershell_7.4.1-1.deb_amd64.deb && sudo apt-get install -f

# RHEL 8
sudo dnf install https://github.com/PowerShell/PowerShell/releases/download/v7.4.1/powershell-7.4.1-1.rh.x86_64.rpm

# MacOS
brew install --cask powershell

# Docker
docker run -it mcr.microsoft.com/powershell
```

See [Installing PowerShell](https://learn.microsoft.com/en-us/powershell/scripting/install/installing-powershell?view=powershell-7.4)

### A good standard library, even better with .NET Core

Powershell comes with all of the standard functions that you expect, but it also have some neat extras that you in most other languages need to use a module for.

#### Easy to work with HTTP and REST API's

```powershell
# Invoke-WebRequest or iwr for short
PS> Invoke-WebRequest -Uri "https://date.nager.at/api/v2/publicholidays/2024/NO"

StatusCode        : 200
StatusDescription : OK
Content           : [{"date":"2024-01-01","localName":"Første nyttårsdag","name":"New Year's Day","countryCode":
                    "NO","fixed":false,"global":true,"counties":null,"launchYear":null,"type":"0"},{"date":"2024
                    -03-28","localNa…
RawContent        : HTTP/1.1 200 OK
                    Date: Fri, 05 Apr 2024 11:35:14 GMT
                    Transfer-Encoding: chunked
                    Connection: keep-alive
                    Cache-Control: public, max-age=604800
                    CF-Cache-Status: HIT
                    Age: 239713
                    Report-To: {"endpoints":[{"…
Headers           : {[Date, System.String[]], [Transfer-Encoding, System.String[]], [Connection,
                    System.String[]], [Cache-Control, System.String[]]…}
Images            : {}
InputFields       : {}
Links             : {}
RawContentLength  : 2000
RelationLink      : {}
```

```powershell
# Invoke-RestMethod or irm for short
PS> Invoke-RestMethod -Uri "https://date.nager.at/api/v2/publicholidays/2024/NO" | ft

date       localName             name              countryCode fixed global counties launchYear type
----       ---------             ----              ----------- ----- ------ -------- ---------- ----
2024-01-01 Første nyttårsdag     New Year's Day    NO          False   True                     0
2024-03-28 Skjærtorsdag          Maundy Thursday   NO          False   True                     0
2024-03-29 Langfredag            Good Friday       NO          False   True                     0
2024-03-31 Første påskedag       Easter Sunday     NO          False   True                     0
2024-04-01 Andre påskedag        Easter Monday     NO          False   True                     0
2024-05-01 Første mai            Labour Day        NO          False   True                     0
2024-05-09 Kristi himmelfartsdag Ascension Day     NO          False   True                     0
2024-05-17 Syttende mai          Constitution Day  NO          False   True                     0
2024-05-19 Første pinsedag       Pentecost         NO          False   True                     0
2024-05-20 Andre pinsedag        Whit Monday       NO          False   True                     0
2024-12-25 Første juledag        Christmas Day     NO          False   True                     0
2024-12-26 Andre juledag         St. Stephen's Day NO          False   True                     0
```

With `Invoke-WebRequest` you get full control of the response, but most times you interact with REST API's using JSON, and `Invoke-RestMethod` will automatically convert the response JSON to a Powershell object. Very handy!

#### Built-in support for reading and writing different formats

```powershell
PS> Get-Command -Module Microsoft.PowerShell* ConvertTo-*

CommandType     Name                                               Version    Source
-----------     ----                                               -------    ------
Cmdlet          ConvertTo-Csv                                      7.0.0.0    Microsoft.PowerShell.Utility
Cmdlet          ConvertTo-Html                                     7.0.0.0    Microsoft.PowerShell.Utility
Cmdlet          ConvertTo-Json                                     7.0.0.0    Microsoft.PowerShell.Utility
Cmdlet          ConvertTo-SecureString                             7.0.0.0    Microsoft.PowerShell.Security
Cmdlet          ConvertTo-Xml                                      7.0.0.0    Microsoft.PowerShell.Utility

PS> Get-Command -Module Microsoft.PowerShell* ConvertFrom-*

CommandType     Name                                               Version    Source
-----------     ----                                               -------    ------
Cmdlet          ConvertFrom-Csv                                    7.0.0.0    Microsoft.PowerShell.Utility
Cmdlet          ConvertFrom-Json                                   7.0.0.0    Microsoft.PowerShell.Utility
Cmdlet          ConvertFrom-Markdown                               7.0.0.0    Microsoft.PowerShell.Utility
Cmdlet          ConvertFrom-SecureString                           7.0.0.0    Microsoft.PowerShell.Security
Cmdlet          ConvertFrom-StringData                             7.0.0.0    Microsoft.PowerShell.Utility
```

All the `ConvertTo-*` cmdlets takes a Powershell object and return as the specified format and all the `ConvertFrom-*` cmdlets will return data into a Powershell object that makes it easier for us to work with. For example you can use `ConvertTo-Html` or `ConvertTo-Csv` to easily create reports.

You can even go further and install the `ImportExcel` module by [Doug Fink](https://www.powershellgallery.com/profiles/DougFinke) from [Powershell Gallery](https://www.powershellgallery.com/packages/ImportExcel), which will give you full access to converting, exporting and importing data between Excel and Powershell!

### Good support for major cloud and on-premise providers (Azure, Amazon, VMWare etc) through robust modules

It should be no suprise that Azure has great module support in Powershell for working with Azure and Microsoft services. But both Amazon (with its AWSPowershell module) and VMWare (PowerCLI module) has great support for Powershell. Google also has its own Powershell module (GoogleCloud), but it's not maintained anymore. They have created their own CLI app.

There are many other providers that has support for Powershell. Take a look at [Powershell Gallery](https://powershellgallery.com).

### Is fit for both professional developers and operations personel

Powershell is syntactically easy to learn, read and write and will feel familiar for both professional developers and operations personel (who probably is already using Powershell for some tasks). Both groups can contribute code. Essentially perfect for DevOps.

### Both a programming language and a shell (like bash, but easier and more readable)

Powershell is both a programming language and a shell, which means that you can write programs with it but also navigate and maintain your computer with it. I spend most of my working day in the terminal using Powershell, navigating around, editing files, running small utility functions to do my job. I also have larger projects/programs written in Powershell that are running in production in Azure. It's the same language but for multiple usecases.

### Object-oriented

Unlike bash, were you are mostly working with plain text, most of the things you touch in Powershell are objects. This makes is really easy to work with JSON or CSV data, pipeing data between functions, manipulating strings etc..

### You have the full power of .NET Core if you choose so, but you dont have to

Since Powershell is built on top of .NET Core, you have access to all the functions and features of .NET Core. If there are things you dont find in Powershell you can directly access classes and functions from .NET Core (which is what many C# developers use) and ta-da you have one of the largest libraries available to you. It's really awesome and extremly handy!

This way you can quickly write out a script using only Powershell, or create larger programs that is more reliant on performance using functions and classes from .NET Core. It's no secret that pure Powershell isn't the fast language out there. But it doesn't have to be either.

### Support for concurrency

The concurrency support in Powershell is not great, BUT it is supported. You can take any piece of code and make it run asynchronously with `Start-Job`. Starting with Powershell 7.2 a new argument is added to `ForEach-Object` that lets you run the body in parallel.

I hope we can see better support for this in the future!

### Tooling support

- Syntax highlighting
- Intellisens
- Formatter (PSScriptAnalyzer)
- Linter (PSScriptAnalyzer)
- Powershell extension for VS Code
- Standalone PoweshellEditorServices language server for Powershell for use with other editors than VSCode (i.e neovim)


## Further exploring

If this article made you want to take a closer look at Powershell, here are some good resources and communities.

- [Discord: Powershell](https://discord.com/invite/powershell)
- [Reddit: Powershell](https://reddit.com/r/powershell)
- [Powershell DevBlog](https://devblogs.microsoft.com/powershell/)
- [Microsoft Learn: Introduction to Powershell](https://learn.microsoft.com/en-us/training/modules/introduction-to-powershell/)
- [Microsoft Learn: Automate administrative tasks by using PowerShell](https://learn.microsoft.com/en-us/training/paths/powershell/)
- [PowerShell Explained with Kevin Marquette](https://powershellexplained.com)
- [John Savill's Technical Training: Powershell Master Class](https://www.youtube.com/playlist?list=PLlVtbbG169nFq_hR7FcMYg32xsSAObuq8)
- [Oh My Posh](https://ohmyposh.dev/)
