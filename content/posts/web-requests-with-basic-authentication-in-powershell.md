---
title: 'Web requests with basic authentication in Powershell'
description: "HTTP Basic Authentication is one of many authentication schemes supported by the HTTP protocol, and is a very common option when authenticating to a web service. Let's explore some examples in Powershell."
date: 2022-07-05T00:00:00
tags: ['Powershell', 'Basic Authentication']
image: "web-requests-with-basic-authentication-in-powershell.svg"
---

HTTP Basic Authentication is one of many authentication schemes supported by the HTTP protocol, and is a very common option when authenticating to a web service. The basic authentication scheme is very simple and consists of generating a base64 token from your username and password seperated by a colon (`:`) and putting the token in an `Authorization` HTTP header. Let's explore some examples in Powershell.

## Manually creating the token

Let's start with an example from scratch.

```powershell
# We define our username and password. Ideally this should come from environment variables
# or some secret store
$username = "user1"
$password = "pa55w0rd!"

# Join them into a single string, seperated by a colon (:)
$pair = "{0}:{1}" -f ($username, $password)

# Turn the string into a base64 encoded string
$bytes = [System.Text.Encoding]::ASCII.GetBytes($pair)
$token = [System.Convert]::ToBase64String($bytes)

# Define a basic 'Authorization' header with the token
$headers = @{
    Authorization = "Basic {0}" -f ($token)
}

# Send a web request using our authorization header
$response = Invoke-RestMethod -Uri "https://example.com/api" -Headers $headers
```

As you can see from the example above, we take our username and password and combine them into a single string seperated by a colon (`:`). Then we take that string and turn it into a Base64 encoded string. This is our token that we need to pass into the `Authorization` header. Our token will look like this.

```plaintext
dXNlcjE6cGE1NXcwcmQh
```

Line 14-16 is were we create a custom header object to send with our request. Here we define the `Authorization` header and we tell it to use `Basic` authorization and then we provide our token.

On the last line we send our request with the custom header.

## The powershell way

Since Basic Authentication is so common, Powershell has of course implemented a simpler solution.

```powershell
# Again, these should come from env vars, Key Vault or some other secret store
$username = "user1"
$password = "pa55w0rd!"

# Since our password is plaintext we must convert it to a secure string
# before creating a PSCredential object
$securePassword = ConvertTo-SecureString -String $password -AsPlainText
$credential = [PSCredential]::new($username, $securePassword)

# Tell Invoke-RestMethod to use Basic Authentication scheme with our credentials
$response = Invoke-RestMethod -Uri "https://example.com/api" -Authentication Basic -Credential $credential
```

This works by telling the `Invoke-RestMethod` cmdlet which authentication scheme we want to use and provide a `PSCredential` object and it will do the rest for us.

This way is much simpler because we dont need to worry about generating the token and in many situations we already have a `PSCredential` object.

## Conclusion

Knowing how to use basic authentication with Powershell can be very handy since most systems support this authentication scheme. As we saw in this article Powershell has made it really simple to use.

## Resources

- [RFC 7617](https://datatracker.ietf.org/doc/html/rfc7617)
- [Wikipedia: Basic access authentication](https://en.wikipedia.org/wiki/Basic_access_authentication)
- [Reference: Invoke-RestMethod](https://docs.microsoft.com/en-us/powershell/module/microsoft.powershell.utility/invoke-restmethod?view=powershell-7.2)
