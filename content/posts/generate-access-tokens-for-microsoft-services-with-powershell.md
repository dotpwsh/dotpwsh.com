---
title: "Generate Access Tokens for Microsoft Services With Powershell"
description: "As automators we often need to interact with REST API's and if you are working with Microsoft Azure you probably found yourself dealing with several of Microsoft's services i.e Microsoft Graph, Azure Resource Manager or Partner Center. In this article we're going to take a look at two flows for how we can authenticate with the different services."
date: 2022-02-09T15:50:44+01:00
tags: ["Powershell", "REST API", "Microsoft"]
image: "generate-access-tokens-for-microsoft-services-with-powershell.svg"
---

As automators we often need to interact with REST API's and if you are working with Microsoft Azure you probably found yourself dealing with several of Microsoft's services i.e Microsoft Graph, Azure Resource Manager or Partner Center. Many of these services is supported by a Powershell module that handles authentication etc.. But I have found lately that more often than not it's actually easier to just work with the raw REST API, especially for cross-platform development. In this article we're going to take a look at two flows for how we can authenticate with the different services.

## The baseline

The baseline for all requests for an access token to Microsoft services is this:

* The URL is `https://login.microsoftonline.com/<tenant_id>/oauth2/v2.0/token`
* The Method is `POST`
* The Content-Type is `application/x-www-form-urlencoded`
* The request body requires these parameters
  * client_id
  * client_secret
  * grant_type
  * scope

In the response you will get an `access_token` which you include in the request header as `Authorization: Bearer <access_token>` in subsequent requests.

## Client Credentials

This is the flow you most likely will use if you are authenticating as a service principal. For the client credentials flow we need to set `grant_type` to `client_credentials`.

```powershell
$reqBody = @{
    client_id     = "<client_id>"
    client_secret = "<client_secret>"
    grant_type    = "client_credentials"
    scope         = "https://graph.microsoft.com/.default"
}

$params = @{
    Uri         = "https://login.microsoftonline.com/{0}/oauth2/v2.0/token" -f ("<tenant_id>")
    Method      = "POST"
    ContentType = "application/x-www-form-urlencoded"
    Body        = $reqBody
}

$token = Invoke-RestMethod @params
```

## Refresh Token

This is the flow you most likely will use if you need to authenticate on behalf of a user that requires MFA. In addition to changing `grant_type` to `refresh_token` we also need to provide a refresh token.

```powershell
$reqBody = @{
    client_id     = "<client_id>"
    client_secret = "<client_secret>"
    grant_type    = "refresh_token"
    refresh_token = "<your_refresh_token>"
    scope         = "https://graph.microsoft.com/.default"
}

$params = @{
    Uri         = "https://login.microsoftonline.com/{0}/oauth2/v2.0/token" -f ("<tenant_id>")
    Method      = "POST"
    ContentType = "application/x-www-form-urlencoded"
    Body        = $reqBody
}

$token = Invoke-RestMethod @params
```

## Scopes

So, how do we use this to authenticate to the different services? Up until now we only authenticated with Microsoft Graph, meaning that our `access_token` will only work with request to Microsoft Graph REST API. To authenticate with other services we simply change the scope. Here are some usefull scopes:

* https://graph.microsoft.com/.default (Microsoft Graph API)
* https://management.azure.com/.default (Azure Resource Manager API)
* https://api.partnercenter.microsoft.com/user_impersonation (Partner Center REST API, requires a refresh_token)

## Wrap-Up

Micrsoft supports several authentication flows and you can read more indept about them here: [MSAL Authentication Flows](https://docs.microsoft.com/en-us/azure/active-directory/develop/msal-authentication-flows).
