---
title: "Sign and verify data with ECDSA in Powershell"
description: "How to sign and verify data with ECDSA cryptography in Powershell"
date: 2024-06-27T15:34:33+02:00
tags: ["powershell", "cryptography", "ecdsa"]
---

## 1. Generate a ECDSA private/public key using openssl

Generate private key

```console
openssl ecparam -name prime256v1 -genkey -noout -out private.pem
```

Generate a public key from our private key

```console
openssl ec -in private.pem -pubout -out public.pem
```

## 2. Strip out header, footer and decode base64

The `ImportECPrivateKey` method expects the key without the header, footer and as decoded base64 byte[].

```powershell
$privkey = Get-Content private.pem
$privkey = $privkey.Replace("-----BEGIN EC PRIVATE KEY-----", "")
$privkey = $privkey.Replace("-----END EC PRIVATE KEY-----", "")
$privkey_decoded = [System.Convert]::FromBase64String($privkey)
```

## 3. Import ECDSA private key

Let's import our decoded private key.

```powershell
$sender = [System.Security.Cryptography.ECDsa]::Create()
$sender.ImportECPrivateKey($privkey_decoded, [ref]$null)
```

## 4. Sign data

First we initialize some data in a string, then we convert it to an byte[]. Then we sign the data using our private key and the SHA256 hashing algorithm.

```powershell
$data = "Hello, World"
$bytes = [System.Text.Encoding]::UTF8.GetBytes($data)
$signed_data = $sender.SignData($bytes, [System.Security.Cryptography.HashAlgorithmName]::SHA256)
```

## 5. Import public key

```powershell
$pubkey = Get-Content public.pem
$pubkey = $pubkey.Replace("-----BEGIN PUBLIC KEY-----", "")
$pubkey = $pubkey.Replace("-----END PUBLIC KEY-----", "")
$pubkey_decoded = [System.Convert]::FromBase64String($pubkey)

$receiver = [System.Security.Cryptography.ECDsa]::Create()
$receiver.ImportSubjectPublicKeyInfo($pubkey_decoded, [ref]$null)
```

## 6. Verify data

We can verify that the data has not been tampered with by comparing the original data and the signed data using our key. This would typically be done on the "other" side using our public key.

```powershell
$receiver.VerifyData($bytes, $signed_data, [System.Security.Cryptography.HashAlgorithmName]::SHA256)
```

This should print "True" to the console.
