---
title: 'Enable External Users to Book Meeting Rooms'
date: 2020-02-14T00:00:00
tags: ['Exchange Online', 'Powershell']
---

From time to time you might need to share one or more of your meeting rooms with external users. In this posts we're going through three options for how you can enable this. Other options do exists (like connecting an Office 365 tenant to another and so), but this post is meant for the time you need to give temporary access or only to one or more users.

_Disclaimer: I have only tried this is Exchange Online._

## Enable processing of external meeting messages

For Exchange Online to even consider accepting external messages we must enable `ProcessExternalMeetingMessages` via Powershell.

```powershell
Set-CalendarProcessing -Identity "Meeting Room Name" `
-ProcessExternalMeetingMessages $true
```

This will open up for EVERYONE to be able to book the meeting room. Most likely you'd want to limit it to either some users or domains. We'll take a look at that next.

## Limit external users

There are a few ways we can limit which of our external users that can book the meeting room.

* Mail-enabled Security Groups
* Transport Rules
* Set-CalendarProcessing

### 1. Mail-enabled Security Groups


> NOTE: I assume you already have either a distribution group or mail-enabled security group for all your internal users or know how to create one.

The first solution we're looking at is mail-enabled security groups in Office 365. Most likely you already have some sort of distribution group or mail-enabled security group with all of your internal users. In addition we're going to add another mail-enabled security group for our external users. This is only viable if the amount of users is not too high, because you must add/remove them manually.

Now, this is **IMPORTANT**. You must create the mail-enabled security group in Office 365 directly. If you have a hybrid setup and create the group in Active Directory, you will not be able to add the external user later.

#### Create group in Office 365

1. Go to **Groups -> Add a group**
2. Choose **Mail-enabled security**
3. Name your group something appropriate (e.g "Meeting Room External Booking") and give it a description
4. Give it an email address
5. Click on **Create group**

#### Set BookInPolicy on meeting room

Our next step is to add the group we just created to the `BookInPolicy` for our meeting room.

Remember I assumed you had a group for all your internal users? Well, we're going to use it now (in this example we'll call it "Company Internal Users"). If your meeting room does not already have a BookInPolicy we must add both groups (internals and externals) and set `AllBookInPolicy` to false.


```powershell
Set-CalendarProcessing -Identity "Meeting Room Name" `
-AllBookInPolicy $false `
-BookInPolicy "Company Internal Users", "Meeting Room External Booking"
```

This will set both groups as BookInPolicy meaning that only users that is member of either groups are allowed for booking. Everyone else will get a rejected message.

#### Add external user to our group

Let's finish off by adding our external users. We do so by invited them in Azure Active Directory as guests and then add them to the mail-enabled security group we created earlier.


1. Go to **Azure Active Directory -> Users**
2. Click on **New guest user**
3. Make sure **Invite user** is checked
4. Fill in necessary information
    1. **Name:** DisplayName
    2. **Email Address:** The guest's external email address
    3. **Groups:** Unfortunately you cannot select our group from here, so leave this as default
    4. **Role:** User
    5. **Block sign in:** Must be "No"
5. Click on **Invite**
6. Go back to **Office 365 -> Groups**
7. Click on "Meeting Room External Booking" and go to **Members** tab
8. Click **View and manage members**
9. Click **+ Add members**
10. Select your invited guest(s) in the list and click **Save**

They should now be able to book the meeting room this was setup for, even though they are external.

### 2. Transport Rules

> NOTE: This only works if `AllBookInPolicy` is true and `BookInPolicy` is empty.

Since we enabled `ProcessExternalMeetingMessages` everyone has access to book our meeting room. Instead of limited whom by using groups we can create a transport rule that states something in the line of: "If email is to this meeting room and is from outside, delete it. Unless its from @domain.com/user@domain.com".

#### Powershell

This can easily be achieved with Powershell. Change `SentTo` and `ExceptIfSenderDomainIs` with your values.

```powershell
New-TransportRule -Name "Meeting Room External Booking" `
-SentTo "meeting_room@contoso.com" `
-ExceptIfSenderDomainIs external-corp.com `
-FromScope NotInOrganization `
-DeleteMessage $true
```

#### Exchange Admin Center

If you're not comfortable with Powershell, you can do it in the Exchange Admin Center in Office 365.

1. Choose **mail flow** in the menu to the left
2. Make sure you're on the **rules** tab
3. Click the **+**-symbol and choose **Create a new rule**

![External Booking Transport Rule](/images/external-booking-transport-rule.png)

Save and you're done.

### 3. Set-CalendarProcessing

Our last option is to set the external user(s) directly in Powershell with `Set-CalendarProcessing`.


> NOTE: This does not support whole domains only single users, just like using a group.

What we will do is to add our external user(s) to the BookInPolicy explicit instead of using a group like in the first option. Using a group is a lot more mangeable since you can just add/remove user(s) in the group and it also give better visibility. But if you for some reason wont use a group, this is how to set it with set-calendarprocessing.

Note that we must disable `AllBookInPolicy` for the `BookInPolicy` to be activated. That means you have to add a group of everyone else that also should be able to book our meeting room, just like in option 1 where we added "Company Internal Users".

```powershell
Set-CalendarProcessing -Identity "Meeting Room Name" `
-AllBookInPolicy $false `
-BookInPolicy "Company Internal Users", "user@external-corp.com"
```

Now user@external-corp.com should be able to book our meeting room.

## Conclusion

We have looked at three options for how we can allow external users to book our meeting room. Out of these options, I would recommend option 1 if there's only specific users. Options 2 would be good for allowing whole domains. Options 3 is just an option. It works, but it does not have any benefits over the others, which are easier to manage.
