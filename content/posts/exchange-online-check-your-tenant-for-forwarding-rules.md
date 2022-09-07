---
title: 'Exchange Online: Check your tenant for forwarding rules'
description: "In this guide we’ll take a look at how you can scan your tenant for “hidden” forwarding rules by using Powershell with Exchange Online."
date: 2018-12-07T00:00:00
tags: ['Powershell', 'Office 365']
---

In this guide we’ll take a look at how you can scan your tenant for “hidden” forwarding rules by using Powershell with Exchange Online.

## Why

One technique that is common amongs hackers, that gain access to email accounts, is to setup a forwarding rule for all incomming email. That way they can read all new emails sent to the victim without beeing flagget or detected by audit logs. They can also create rules for emails from a specific address that go to a folder the hacker controls, ie password reset emails. They can then request password change for a numerous different services without the user beeing alerted right away and maybe get further into your system. Especially if its cloud based.

Or, it could be an unfaithful employee that forward emails to a competitor. Either way its critical.

## How

Exchange Online provides two methods for creating forwarding rules; Inbox Settings and Inbox Rules.

### Inbox Settings

In Exchange Online there’s a specific setting that activates forwarding. In OWA (Outlook Web App) you can find it by browsing to **Mail → Accounts → Forwarding**. In EAC (Exchange Admin Center) open user mailbox then go to **Mailbox Features → Mail Flow → Delivery Options**.

### Inbox Rules

> A rule is an action that Outlook Web App runs automatically on incoming or outgoing messages. For example, you can create a rule to automatically move all email sent to a group you are a member of to a specific folder, or to delete all messages with “Buy now” in the subject.

This is how Microsoft referers to inbox rules. Basically they are rules that are executed on incomming email and we are going to look for those who does forwarding.

## How to scan your tenant

First, let’s scan for inbox settings.

```powershell
Get-Mailbox -ResultSize unlimited |
Where-Object { ($_.ForwardingSMTPAddress -ne $null) -or ($_.ForwardingAddress -ne $null) } |
Select-Object Name, ForwardingSMTPAddress, ForwardingAddress, DeliverToMailboxAndForward
```

**Line 1:** We collect all mailboxes in our tenant. ResultSize is by default 1000, so if your have more mailboxes you must include -ResultSize unlimited.

**Line 2:** We will filter out mailboxes which does not have the ‘ForwardingSMTPAddress’ or ‘ForwardingAddress’ attribute set. In other words email forwarding is not configured through settings.

Now let’s scan for inbox rules, which is a little more complicated.

```powershell
$Mailboxes = Get-Mailbox -ResultSize unlimited
ForEach ($Mailbox in $Mailboxes) {
    $MailboxWithRule = Get-InboxRule -Mailbox $Mailbox.Alias | Where-Object { ($_.RedirectTo -ne $null) -or ($_.ForwardTo -ne $null) -or ($_.ForwardAsAttachmentTo -ne $null) }
    If ($MailboxWithRule -ne $Null) {
        Write-Host "Mailbox $($Mailbox.PrimarySmtpAddress) has the following rules configured:"
        $MailboxWithRule | Format-List Name, Identity, RedirectTo, ForwardTo, ForwardAsAttachmentTo
    }
}
```

**Line 1:** We collect all mailboxes in our tenant. ResultSize is by default 1000, so if your have more mailboxes then that you must include -ResultSize unlimited.

**Line 2:** We loop through each mailbox.

**Line 3:** We fetch all inbox rules for current mailbox and filter out those who does not contain a RedirectTo, ForwardTo or ForwardAsAttachment value. _NOTE: This cmdlet is pretty slow._

**Line 4-7:** If line 3 returns any rules, we will print the address of the mailbox and return all the rules we found.

The output looks a little something like this:

```plaintext
PS C:\Users\dotsh> .\ExportExchangeForwardingInboxRules.ps1
...
Mailbox user@contoso.com has the following rules configured:

Name                  : SilentForwarding
PrimarySmtpAddress    : user@contoso.com
Identity              : Example User 1\8573016583648562395
RedirectTo            :
ForwardTo             : {"Malicous User" [SMTP:hacker@l33t.com]}
ForwardAsAttachmentTo :

Name                  : MyRule
PrimarySmtpAddress    : user@contoso.com
Identity              : Example User 1\8573016583648562395
RedirectTo            :
ForwardTo             : {"Example User 2" [SMTP:user2@contoso.com], "Example User 3" [SMTP:user3@contoso.com],
                        "Example User 4" [SMTP:user4@contoso.com]}
ForwardAsAttachmentTo :

...
```

## Finishing up

Often you would forward these results to head of security or something. So to make it easier for them to read the report we could export it to an CSV thats opens up in Excel. The easiest way to do this is to put each script in its own file, ie `ExportExchangeForwardingInboxSettings.ps1` and `ExportExchangeForwardingInboxRules.ps1`. Then we could call it like this to export it to CSV.

```plaintext
PS C:\Users\dotsh> .\ExportExchangeForwardingInboxSettings.ps1 | Export-Csv -Encoding UTF8 -Delimiter ";" -NoTypeInformation -Path "ExchangeForwardingInboxSettings-Report.csv"
...
PS C:\Users\dotsh> .\ExportExchangeForwardingInboxRules.ps1 | Export-Csv -Encoding UTF8 -Delimiter ";" -NoTypeInformation -Path "ExchangeForwardingInboxRules-Report.csv"
...
```

By using `-Delimiter ';'` Excel will automatically format the table correctly for us.

Now its just for you to send the report to the right person and you may have avoided a massive data breach.
