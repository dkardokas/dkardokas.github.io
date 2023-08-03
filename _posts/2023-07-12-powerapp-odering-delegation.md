---
title: Power App ordering delegation
author: dkardokas
date: 2023-07-12 09:55:00 +0100
categories: [Power Apps]
tags: [Power Platform, Delegation, Data connections]
---

You may be familiar with the infamous [delegation warning](https://learn.microsoft.com/en-us/power-apps/maker/canvas-apps/delegation-overview) in Canvas Apps - however if you do not see the warning, that does not necessarily mean you're in the clear.

In this example we're connecting to a SharePoint list, but delegation works similarly with other data sources. Let's say we want to filter data based on one field value, and sort in descending order. 
![formula1](/assets/img/Screenshot 225330.png)

We're using `Filter` and `SortByColumns` functions - both supporting "full delegation" for SharePoint

If our dataset has more than 500 items (or whatever the app data limit is set to), we'll get partial results - but this is fine as we're getting most recent items, as requested by `SortByColumns` function. We can see this in action by inspecting the request being made to SharePoint in a Monitoring session

![monitor1](/assets/img/Screenshot230609.png)

However if the logic is a little more complicated, the delegation falls apart:

![formula2](/assets/img/Screenshot230958.png)

We're still using the same data retrieval functions, however the app/connector no longer sends the `oderby` parameter to SharePoint - instead the sorting is done client-side and we are getting the ~first~ 500 items from the list rather than last as requested (no delegation warning in sight!)

![monitor2](/assets/img/Screenshot 231612.png)

We can help the engine understand the formula better by keeping the data functions close together - which in this case means repeating the sort function for each case

![formula3](/assets/img/Screenshot232318.png)

This resolves the issue and `oderby` parameter is included in the request again 

![monitor2](/assets/img/Screenshot232527.png)

> There may be similar scenarios where seemingly "delegable" functions work inconsistently - inspecting request in a Monitoring session should help find the cause

Hope this helps, if you have similar experience or queries, drop a comment below :point_down:
