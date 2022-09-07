---
title: 'Powershell: Uploading files with multipart/form-data'
description: "By default, when you use the `-InFile` parameter of `Invoke-RestMethod` Powershell will send it as `application/octet-stream`. In this short article we'll look at how to send that file with `multipart/form-data` instead which is what most forms on the web use."
date: 2021-04-20T00:00:00
tags: ['Powershell', 'Confluence']
---

By default, when you use the `-InFile` parameter of `Invoke-RestMethod` Powershell will send it as `application/octet-stream`. In this short article we'll look at how to send that file with `multipart/form-data` instead which is what most forms on the web use.

To send `multipart/form-data` with Powershell we can use the `-Form` parameter which takes a hashtable of fields and values.

I couldn't find a service that would allow us to test file uploads for free, so in this example we're going to use the Confluence Cloud REST API to upload an attachment to a page.

**TIP:** If you want to test out `multipart/form-data` without file uploads httpbin has an endpoint for that: [https://httpbin.org/forms/post](https://httpbin.org/post). Just skip the authentication section and replace the url with this and the form object to reflect the fields on httpbin.

## Authentication

Since we'll be using the Confluence Cloud REST API, we must setup our authentication. Since this is not the main part i'll show you a quick and simple way. If you want more information you can read more here: [Confluence Cloud - Authentication and authorization](https://developer.atlassian.com/cloud/confluence/rest/intro/#auth).

Confluence REST API use Basic Authentication.

```powershell
$BaseURL = "https://<your_atlassian_domain>.atlassian.net"

$credential = Get-Credential
$authString = "{0}:{1}" -f ($Credential.UserName, (ConvertFrom-SecureString $Credential.Password -AsPlainText))
$b64Encoded = [System.Convert]::ToBase64String([System.Text.Encoding]::ASCII.GetBytes($ConfBasicAuthString))
$headers = @{
    "Authorization"     = "Basic $($b64Encoded)"
    "X-Atlassian-Token" = "nocheck"
    "Content-Type"      = "multipart/form-data"
}
```

This is all we need to setup before we can run request against the REST API.

## Uploading an image

Confluence allows us to set three fields in our form:

- file (required)
- comment (optional)
- minorEdit (required)

So, for us to upload a file we must provide the `file` field in our form object. When working with files we can use the `Get-Item` cmdlet to get the file we want to upload.

```powershell
$form = @{
    file      = Get-Item -Path $filename
    minorEdit = "true"
}
```

This is not unique for Confluence. Web-forms that allow file upload will have a field with a `type="file"` and a `name="my_file"`. In your form object you would replace "file" with "my_file".

When we have defined our form, we can submit it with `Invoke-RestMethod` or `Invoke-WebRequest` by setting the `-Form` parameter to our `$form` object.

```powershell
Invoke-RestMethod -Uri "$($BaseURL)/wiki/rest/api/content/{id}/child/attachment" -Method PUT -Headers $headers -Form $form
```

## Confluence spesific caveats

There are some configuration options that are set, but not mentioned.

Confluence itself uses XSRF when posting form-data, so to use this with the REST API we must provide the `X-Atlassian-Token: nocheck` header for it to bypass it.

We also use the `PUT` method instead of `POST` which allows us to add a new attachment if it does not exist, or update it if it already exists.
