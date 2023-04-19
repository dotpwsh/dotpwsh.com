---
title: 'Get type definitions in Powershell'
description: "Today I went back to some Powershell scripting with the Az module and it frustrated me that I wasn't easily able to know what properties `Get-AzADGroup` (or any of the other Az cmdlets) returned to me without actually invoking the cmdlet."
tags: ['Powershell']
date: 2021-08-27T09:43:57+02:00
image: "get-type-definitions-in-powershell.svg"
---

Today I went back to some Powershell scripting with the Az module and it frustrated me that I wasn't easily able to know what properties `Get-AzADGroup` (or any of the other Az cmdlets) returned to me without actually invoking the cmdlet. E.g I dont want to invoke `New-AzADGroup` just to be able to see what properties it will give me so I can use that in my script. Previously I've relied on IntelliSens in my editor, but it often fails, so I sought out to find a more manual solution (who would have thought..).

Some modules provide information in their help about what type a particular cmdlet returns. We can use this to get information about what that type contains.

If we look at the `Get-AzADGroup` cmdlet as an example. When we run `Get-Help Get-AzADGroup -Full` we can see under the "OUTPUT" section that it returns a `Microsoft.Azure.Commands.ActiveDirectory.PSADGroup` type. To inspect that type we can do the following.

```powershell
PS > Get-Help Get-AzADGroup -Full
...

OUTPUTS
    Microsoft.Azure.Commands.ActiveDirectory.PSADGroup

...
PS > [Microsoft.Azure.Commands.ActiveDirectory.PSADGroup]::new() | gm

   TypeName: Microsoft.Azure.Commands.ActiveDirectory.PSADGroup

Name                 MemberType Definition
----                 ---------- ----------
Equals               Method     bool Equals(System.Object obj)
GetHashCode          Method     int GetHashCode()
GetType              Method     type GetType()
ToString             Method     string ToString()
AdditionalProperties Property   System.Collections.Generic.IDictionary[string,System.Object] AdditionalProperties {get;set;}
DeletionTimestamp    Property   System.Nullable[datetime] DeletionTimestamp {get;set;}
Description          Property   string Description {get;set;}
DisplayName          Property   string DisplayName {get;set;}
Id                   Property   string Id {get;set;}
MailEnabled          Property   System.Nullable[bool] MailEnabled {get;set;}
MailNickname         Property   string MailNickname {get;set;}
ObjectType           Property   string ObjectType {get;}
SecurityEnabled      Property   System.Nullable[bool] SecurityEnabled {get;set;}
Type                 Property   string Type {get;set;}

```

Here we can see what methods and properties we will get if we run `Get-AzADGroup`.

Another solution is to call either the `GetMembers()`, `GetProperties()` or `GetMethods()` on the type which will give you detailed information about each member. It's a little verbose to be frank.. So I prefer the first method.

```powershell
PS > [Microsoft.Azure.Commands.ActiveDirectory.PSADGroup].GetProperties()

MemberType       : Property
Name             : SecurityEnabled
DeclaringType    : Microsoft.Azure.Commands.ActiveDirectory.PSADGroup
ReflectedType    : Microsoft.Azure.Commands.ActiveDirectory.PSADGroup
MetadataToken    : 385876641
Module           : Microsoft.Azure.PowerShell.Cmdlets.Resources.dll
IsCollectible    : False
PropertyType     : System.Nullable`1[System.Boolean]
Attributes       : None
CanRead          : True
CanWrite         : True
IsSpecialName    : False
GetMethod        : System.Nullable`1[System.Boolean] get_SecurityEnabled()
SetMethod        : Void set_SecurityEnabled(System.Nullable`1[System.Boolean])
CustomAttributes : {}

MemberType       : Property
Name             : MailEnabled
DeclaringType    : Microsoft.Azure.Commands.ActiveDirectory.PSADGroup
ReflectedType    : Microsoft.Azure.Commands.ActiveDirectory.PSADGroup
MetadataToken    : 385876642
Module           : Microsoft.Azure.PowerShell.Cmdlets.Resources.dll
IsCollectible    : False
PropertyType     : System.Nullable`1[System.Boolean]
Attributes       : None
CanRead          : True
CanWrite         : True
IsSpecialName    : False
GetMethod        : System.Nullable`1[System.Boolean] get_MailEnabled()
SetMethod        : Void set_MailEnabled(System.Nullable`1[System.Boolean])
CustomAttributes : {}

MemberType       : Property
Name             : MailNickname
DeclaringType    : Microsoft.Azure.Commands.ActiveDirectory.PSADGroup
ReflectedType    : Microsoft.Azure.Commands.ActiveDirectory.PSADGroup
MetadataToken    : 385876643
Module           : Microsoft.Azure.PowerShell.Cmdlets.Resources.dll
IsCollectible    : False
PropertyType     : System.String
Attributes       : None
CanRead          : True
CanWrite         : True
IsSpecialName    : False
GetMethod        : System.String get_MailNickname()
SetMethod        : Void set_MailNickname(System.String)
CustomAttributes : {}

MemberType       : Property
Name             : ObjectType
DeclaringType    : Microsoft.Azure.Commands.ActiveDirectory.PSADGroup
ReflectedType    : Microsoft.Azure.Commands.ActiveDirectory.PSADGroup
MetadataToken    : 385876644
Module           : Microsoft.Azure.PowerShell.Cmdlets.Resources.dll
IsCollectible    : False
PropertyType     : System.String
Attributes       : None
CanRead          : True
CanWrite         : False
IsSpecialName    : False
GetMethod        : System.String get_ObjectType()
SetMethod        :
CustomAttributes : {}

MemberType       : Property
Name             : Description
DeclaringType    : Microsoft.Azure.Commands.ActiveDirectory.PSADGroup
ReflectedType    : Microsoft.Azure.Commands.ActiveDirectory.PSADGroup
MetadataToken    : 385876645
Module           : Microsoft.Azure.PowerShell.Cmdlets.Resources.dll
IsCollectible    : False
PropertyType     : System.String
Attributes       : None
CanRead          : True
CanWrite         : True
IsSpecialName    : False
GetMethod        : System.String get_Description()
SetMethod        : Void set_Description(System.String)
CustomAttributes : {}

MemberType       : Property
Name             : DisplayName
DeclaringType    : Microsoft.Azure.Commands.ActiveDirectory.PSADObject
ReflectedType    : Microsoft.Azure.Commands.ActiveDirectory.PSADGroup
MetadataToken    : 385876650
Module           : Microsoft.Azure.PowerShell.Cmdlets.Resources.dll
IsCollectible    : False
PropertyType     : System.String
Attributes       : None
CanRead          : True
CanWrite         : True
IsSpecialName    : False
GetMethod        : System.String get_DisplayName()
SetMethod        : Void set_DisplayName(System.String)
CustomAttributes : {}

MemberType       : Property
Name             : Id
DeclaringType    : Microsoft.Azure.Commands.ActiveDirectory.PSADObject
ReflectedType    : Microsoft.Azure.Commands.ActiveDirectory.PSADGroup
MetadataToken    : 385876651
Module           : Microsoft.Azure.PowerShell.Cmdlets.Resources.dll
IsCollectible    : False
PropertyType     : System.String
Attributes       : None
CanRead          : True
CanWrite         : True
IsSpecialName    : False
GetMethod        : System.String get_Id()
SetMethod        : Void set_Id(System.String)
CustomAttributes : {}

MemberType       : Property
Name             : Type
DeclaringType    : Microsoft.Azure.Commands.ActiveDirectory.PSADObject
ReflectedType    : Microsoft.Azure.Commands.ActiveDirectory.PSADGroup
MetadataToken    : 385876652
Module           : Microsoft.Azure.PowerShell.Cmdlets.Resources.dll
IsCollectible    : False
PropertyType     : System.String
Attributes       : None
CanRead          : True
CanWrite         : True
IsSpecialName    : False
GetMethod        : System.String get_Type()
SetMethod        : Void set_Type(System.String)
CustomAttributes : {}

MemberType       : Property
Name             : DeletionTimestamp
DeclaringType    : Microsoft.Azure.Commands.ActiveDirectory.PSADObject
ReflectedType    : Microsoft.Azure.Commands.ActiveDirectory.PSADGroup
MetadataToken    : 385876653
Module           : Microsoft.Azure.PowerShell.Cmdlets.Resources.dll
IsCollectible    : False
PropertyType     : System.Nullable`1[System.DateTime]
Attributes       : None
CanRead          : True
CanWrite         : True
IsSpecialName    : False
GetMethod        : System.Nullable`1[System.DateTime] get_DeletionTimestamp()
SetMethod        : Void set_DeletionTimestamp(System.Nullable`1[System.DateTime])
CustomAttributes : {}

MemberType       : Property
Name             : AdditionalProperties
DeclaringType    : Microsoft.Azure.Commands.ActiveDirectory.PSADObject
ReflectedType    : Microsoft.Azure.Commands.ActiveDirectory.PSADGroup
MetadataToken    : 385876654
Module           : Microsoft.Azure.PowerShell.Cmdlets.Resources.dll
IsCollectible    : False
PropertyType     : System.Collections.Generic.IDictionary`2[System.String,System.Object]
Attributes       : None
CanRead          : True
CanWrite         : True
IsSpecialName    : False
GetMethod        : System.Collections.Generic.IDictionary`2[System.String,System.Object] get_AdditionalProperties()
SetMethod        : Void set_AdditionalProperties(System.Collections.Generic.IDictionary`2[System.String,System.Object])
CustomAttributes : {}

```
