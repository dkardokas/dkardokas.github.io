---
title: Guest Account access to Power Apps
author: dkardokas
date: 2023-08-13 09:55:00 +0100
categories: [Power Apps]
tags: [Canvas Apps, Azure AD, Vulnerabilities]
---

A few articles have been published lately discussing the risks Guest accounts pose when combined with access to Power Apps - especially the easy way a guest can bring in their licence, including the free [Developer](https://learn.microsoft.com/en-us/power-apps/maker/signup-for-powerapps) versions (both for AAD and Microsoft accounts).
This seems to have been triggered by a [talk](https://www.blackhat.com/us-23/briefings/schedule/index.html#all-you-need-is-guest-32647) in blackhat USA conference by [Michael Bargury](https://github.com/mbrg) but was then picked up by [The Register](https://www.theregister.com/2023/08/10/microsoft_365_guest_accounts_power/?td=rt-9cp) and [others](https://www.techzine.eu/news/security/109979/guest-accounts-microsoft-365-lead-to-major-cyber-threat/)

While there's arguably no groundbreaking revelations in the talk itself - we already know that users, including guests, have permission to create Power Apps by default - it highlights the importance of adhering simple 'common sense' practices
- Do not share Power Platform connectors except specific scenarios (e.g. specific service identities)
- Have a policy for dealing with Guest accounts - specific reasons for inviting, reviewing after certain period, require MFA
- Regularly review pending invitations - it is better to cancel and invite again than let an in invitation hang for a long period

Here's a couple of useful snippets helpful in these scenarios ([Azure AD](https://learn.microsoft.com/en-us/powershell/module/azuread/?view=azureadps-2.0) module is required for these to work)

Find current Guest users:
```powershell

Connect-AzureAD

# Get all Guest users in the tenant

Get-AzureADUser -All $true | where {$_.userPrincipalName -like "#EXT#"}

#alternatively

Get-AzureADUser -All $true -filter "usertype eq 'guest'"

```

See all pending invitations:

```powershell
Connect-AzureAD 
 
#Get all pending invites
Get-AzureADUser -Filter "UserState eq 'PendingAcceptance'" 

```

This is usually the first step in a more holistic approach - we have worked with clients where the output of these is cross-referenced with other data such as current projects and the parties involved, business requests for external collaboration etc.