---
title: 'Powershell: Extract Windows Spotlight images'
date: 2018-12-12T00:00:00
tags: ['Powershell', 'Windows 10']
---

A very nice feature of Windows 10 is Windows Spotlight who serves beautiful wallpapers on your lock screen every day. It’s a shame these beautiful images are hidden in a system folder somewhere in Windows, so today I’m going to show you how you can extract these images with Powershell. You could perfectly do this manually, but since these images change periodically (haven’t found any info on when) its much easier to just run a script. Personally I run this script as a scheduled job everyday.

So here’s the script:

```powershell
<# Filename: Get-WindowsSpotlightImages.ps1 #>

Param (
    [String] $OutputFolder = "$env:USERPROFILE\Pictures\Spotlight"
)

Add-Type -AssemblyName System.Drawing

If(-not (Test-Path $OutputFolder)) {
    New-Item $OutputFolder -ItemType Directory | Out-Null
}

Get-ChildItem "$env:USERPROFILE\AppData\Local\Packages\Microsoft.Windows.ContentDeliveryManager_cw5n1h2txyewy\LocalState\Assets" | ForEach-Object {

    $NewFilename = $_.Name + ".jpeg"
    If(-not (Test-Path (Join-Path $OutputFolder $NewFilename))) {
        Copy-Item $_.Fullname (Join-Path $OutputFolder $NewFilename)
    }

}

$ImagesToRemove = @()
$AllImages = Get-ChildItem (Join-Path $OutputFolder "*.jpeg")
$AllImages | ForEach-Object {
    $image = New-Object System.Drawing.Bitmap $_.FullName
    If(-not ($image.Width -ge 1920)) {
        $ImagesToRemove += $_.FullName
    } else {
    }
    $image.Dispose()
}

$ImagesToRemove | Remove-Item
```

Now lets break it down.

**Line 3-5:** We set the defualt output folder to a spotlight folder in the user’s picture folder.

**Line 7:** We must import the assembly System.Drawing to be able to read the image size.

**Line 9-11:** We create the output folder if it does not exist.

**Line 13-20:** So here you can see the path where Windows Spotlight images are stored. It contains images for both desktop screens and mobile screens. They are not stored with a file extension, so we give them a _.jpeg_ extension so that we can handle them later. Then we copy them to the output folder with the new extension.

**Line 22-31:** We loop through all images in the output folder matching extension _.jpeg._ Then we look for images that are ment for desktop screens (width larger than 1920px) since we dont want the mobile ones.

> NOTE: If you would like to have images ment for both desktop and mobile, you could just remove everything from line 21 and down and line 7.

The filename of images that do not meet our requirment of `width=1920px` will be added to an array `$ImagesToRemove`. It’s important that we dispose the image object when we’re done reading from it, or else you will get a lot of errors.

**Line 33:** This loops through our `$ImagesToRemove` array and delete images in the output folder that match the filename from our array.

## What now

Since Windows Spotlight update it’s images periodically it would make sense to set this script up as a scheduled job or you could run it manually when you feel like.

Also, this script could probably improve. I’m no Powershell master. If you have some improvements, please comment below. I’m eager to learn!
