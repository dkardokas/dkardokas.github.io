---
title: CAML Query 5000 item limit
author: dkardokas
date: 2023-07-21 09:55:00 +0100
categories: [SharePoint, PowerShell]
tags: [PnP PowerShell, SharePoint, Code]
---

When retrieving items from a SharePoint list or library, we can run into a situation where more than 5000 items are matching the query. SharePoint does not allow this due to view threshold

> Get-PnPListItem: The attempted operation is prohibited because it exceeds the list view threshold.

PnP PowerShell does have a neat workaround where we can simply get ~all~ items from the list and apply the required condition on each one, collecting the results in an array

```powershell
$AllItems = Get-PnPListItem -List $ListName -PageSize 500 -Connection $srcConnection
$MatchingItems = @()
Write-host "Total Number of items:" $($AllItems.Count) -ForegroundColor DarkYellow

ForEach ($Item in $AllItems) {    
    if ($null -eq $Item.FieldValues.MyField -or $Item.FieldValues.MyField.Length -eq 0) {                    
        $MatchingItems += $Item
    }
}
```
... this however feels quite inefficient and should only be used if the CAML approach can't be used. But how exactly do we implement it as a fallback?
By using a `try/catch` block of course. But the key point to remember is to pass `-ErrorAction Stop` parameter, otherwise it's not considered a terminating error and therefore `catch` not triggered.

Complete block may look like this

```powershell
try {
    $MatchingItems = Get-PnPListItem -List $ListName -Query $query -Connection $srcConnection -PageSize 500 -ErrorAction Stop
}
catch {
    $AllItems = Get-PnPListItem -List $ListName -PageSize 500 -Connection $srcConnection
    $MatchingItems = @()
    Write-host "Total Number of items:" $($AllItems.Count) -ForegroundColor DarkYellow

    ForEach ($Item in $AllItems) {    
        if ($null -eq $Item.FieldValues.MyField -or $Item.FieldValues.MyField.Length -eq 0) {                    
            $MatchingItems += $Item
        }
    }
}
```