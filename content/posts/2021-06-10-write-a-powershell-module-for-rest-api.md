---
title: "Write a Powershell module for REST API's"
date: 2021-06-10T00:00:00
tags: ['Powershell']
draft: true
---

There are a lot of great RESTful API's out there on the internet which enables developers to create amazing stuff or automate difficult tasks. Even though Powershell makes it really easy to work with REST API's, in most cases having native Powershell module support is better for the broader community. Then they dont have to worry about the authentication mechanism or which endpoints to use or how to use them. They just need to know which cmdlets to call and what data to pass. In this post I will show you how to wrap and exisiting REST API into a Powershell module.

For our example we're going to implement some of the functionallity in the [AbuseIPDB APIv2](https://docs.abuseipdb.com/#introduction), more specifically the `CHECK`, `BLACKLIST` and `REPORT` endpoints. To test our module you also need to sign up and create an API Key from your [account dashboard](https://www.abuseipdb.com/account/api). I'm going to assume that you have some knowledge of how a Powershell module work. If not, you should check our Kevin Marquette's post [Powershell: Building a Module, one microstep at a time](https://powershellexplained.com/2017-05-27-Powershell-module-building-basics/).

## Project setup

Our setup will look pretty much like any Powershell module. We will create a file for each function and put it into either a `private` or `public` folder, and we will use the `.psm1` file to dot-source them and export the public functions.

Heres an overview of how our project is going to look:

```
.
├── LICENSE
├── PSAbuseIPDB.psd1
├── PSAbuseIPDB.psm1
├── Private
│   └── Invoke-AbuseIPDBRequest.ps1
├── Public
│   ├── Get-AbuseIPDBBlacklist.ps1
│   ├── Initialize-AbuseIPDB.ps1
│   ├── New-AbuseIPDBReport.ps1
│   └── Resolve-AbuseIPDBIPAddress.ps1
└── README.md

2 directories, 9 files
```

As you can see we're putting our public funtions that our users will be able to call, inside the `Public` folder, and we have one internal function that we put into our `Private` folder.

The full source code is available on Github [madsaune/PSAbuseIPDB](https://github.com/madsaune/PSAbuseIPDB).

## Authentication

The first thing our module needs to handle is authentication. We cannot call any of the REST API endpoints without authenticating ourself. We're going to build the authentication logic into the `Invoke-AbuseIPDBRequest` function that we will use in all the other functions when calling the API. But first, we need a way for the user to set their API Key so that they dont have to pass it in every function.

```powershell
# File: Initialize-AbuseIPDB.ps1

function Initialize-AbuseIPDB {
    [CmdletBinding()]
    param (
        [Parameter(Mandatory)]
        [string]
        $APIKey
    )

    $script:APIKey = $APIKey
}
```

This function is really simple. Its only job is to take an API Key and assign it to a special variable that will be available to all other functions in our module.

Now let's define our `./Private/Invoke-AbuseIPDBRequest.ps1` function with some parameters.

```powershell
function Invoke-AbuseIPDBRequest {
    [CmdletBinding()]
    param (
        [Parameter(Mandatory)]
        [string]
        $Endpoint,

        [Parameter()]
        [ValidateSet('GET', 'POST', 'PUT', 'DELETE')]
        [string]
        $Method = 'GET',

        [Parameter()]
        [hashtable]
        $Body
    )

    begin {}
    process {}
    end {}
}
```

If we think about it, which pieces of information do we need to call the AbuseIPDB API? Common between the API endspoints is, we need the URL, maybe some data and content-type, the HTTP method to use and credentials. So we define these are parameters.

* The `$Endpoint` is mandatory, because we always need to know which URL to call.
* The `$Method` will have a default value of "GET", since that most common and users will expect it.
* The `$Body` is optional. According to HTTP spec, GET request should not contain a body, therefor we don't always have to provide it.

Both the `Content-Type` and credentials will always be the same for all requests, so we're going to specify them later.

In the `begin {}` block, we're going to setup what we need to create the request.

```powershell
begin {
    $RequestSplat = @{
        Uri         = 'https://api.abuseipdb.com/api/v2{0}' -f ($Endpoint)
        Method      = $Method
        ContentType = 'application/json'
        Headers     = @{
            Key = $script:APIKey
        }
    }
}
```

1. We hardcode the base URI since thats always going to be the same, and then we add on whats passed in as `$Endpoint`.
2. We set the Method to whats specified in `$Method`.
3. We hardcode the ContentType to 'application/json' since all requests that have a body will be in JSON format.
4. We set our API Key in the header as 'Key: <API_KEY>'. This is where AbuseIPDB API will look for our API Key.

Now we can move on to do the request itself.

```powershell
process {
    try {
        $response = Invoke-RestMethod @RequestSplat
        return $response
    } catch {
        $PSCmdlet.ThrowTerminatingError($_)
    }
}
```

Basically in the `process {}` block we execute the request with our parameters and return the response. If any errors we catch them and throw a terminating error with our exeception. This will allow us to catch the error when using this function in the other functions.

This is our full function:

```powershell
function Invoke-AbuseIPDBRequest {
    [CmdletBinding()]
    param (
        [Parameter(Mandatory)]
        [string]
        $Endpoint,

        [Parameter()]
        [ValidateSet('GET', 'POST', 'PUT', 'DELETE')]
        [string]
        $Method,

        [Parameter()]
        [hashtable]
        $Body
    )

begin {
    $RequestSplat = @{
        Uri         = 'https://api.abuseipdb.com/api/v2{0}' -f ($Endpoint)
        Method      = $Method
        ContentType = 'application/json'
        Headers     = @{
            Key = $script:APIKey
        }
    }
}

process {
    try {
        $response = Invoke-RestMethod @RequestSplat
        return $response
    } catch {
        $PSCmdlet.ThrowTerminatingError($_)
    }
}

    end {}
}
```

The benefit of having it in its own function is that we dont need to write all this code for every request. This also makes it more modular so if we for example later need to modify some data before executing the request, we have one place to do it.

## Check an IP Address

What we have created so far are basically utility functions. One internal and one public to make it easier for ourself and users. Now let's implement the first function that acctually calls the API and retrieves some data.

According to the [documentation](https://docs.abuseipdb.com/#check-endpoint) the `CHECK endpoint` accepts two parameters, `maxAgeInDays` and an IP Address in either v4 or v6 format. So let's create a function that will take these two parameters and call the CHECK endpoint.

```powershell
# File: ./Public/Resolve-AbuseIPDBIPAddress.ps1

function Resolve-AbuseIPDBIPAddress {
    [CmdletBinding()]
    param (
        [Parameter(Mandatory)]
        [string]
        $IPAddress,

        [Parameter()]
        [ValidateRange(1, 365)]
        [int]
        $MaxAgeInDays = 30,

        [Parameter()]
        [switch]
        $VerboseOutput
    )

    begin {}

    process {}

    end {}
}
```

We start off by creating the function skeleton and defining parameters. According to the [documentation](https://docs.abuseipdb.com/#check-endpoint) the `CHECK endpoint` accepts two parameters, `maxAgeInDays` and an IP Address in either v4 or v6 format.

If we look closer at the documentation the `maxAgeInDays` has a default value and a valid range. It's always good practive to implement these in your function as it's more clear to the user whats going on.

* The `$IPAddress` is specified as a string as JSON dont handle IP addresses as a seperate type. We could in our module implement some logic to verify that the IP Address is in correct format to help our users, but thats out of the scope for this post. This [StackOverflow post](https://stackoverflow.com/questions/43336422/validate-ip-address-entered-by-user) has some answers if you want to look into it
* The `$MaxAgeInDays` parameter has a default value of 30, and we make sure that its in the range of 1-365 to be valid against the API.

Since we made most of the request logic in `Invoke-AbuseIPDBRequest` the next section will be pretty short (yay!).

```powershell
begin {
    # TODO: Add logic to verify if valid IP Address format

    $Endpoint = '/check?ipAddress={0}&maxAgeInDays={1}' -f ($IPAddress, $MaxAgeInDays)

    if ($VerboseOutput.IsPresent) {
        $Endpoint += '&verbose'
    }

}

process {
    try {
        $response = Invoke-AbuseIPDBRequest -Endpoint $Uri
    } catch {
        $PSCmdlet.ThrowTerminatingError($_)
    }
}
```

## Error handling
