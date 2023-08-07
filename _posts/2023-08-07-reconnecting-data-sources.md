---
title: Reconnecting Canvas app data sources
author: dkardokas
date: 2023-08-07 09:55:00 +0100
categories: [Power Apps]
tags: [Canvas Apps, Low Code, Solutions]
---

When working with Canvas apps we sometimes need to upgrade an app from being a prototype to a Production-ready product (or more often - the prototype has already grown into a business critical app and now the client is nervous about it (as they should be) ). This typically involves adding the app into a solution and reconnecting data sources from hardcoded values to Environment variables. The prospect of finding every reference to a data source and updating it does not sound overly appealing - however there is a very useful and not well known feature of the Power Apps Studio, whereby this is done automagically. Simply removing a \[hard-coded\] data source and adding another one through an environment variable (as long as they are pointing to the same table/SharePoint list etc.) does 99% of the updating required.

![remove](/assets/img/Screenshot 2023-08-07 235046.png)

> If 'Advanced' option is not available when adding a data source, you may need to turn on 'Improve data source experience and Microsoft Dataverse views' setting.

![Add env variable](/assets/img/image.png)

After re-adding data source (in this case - a SharePoint list) all formulas and controls referencing the data source are updated automatically

![Updated formula](/assets/img/formula7.png)
