---
title: SharePoint document in an Iframe
author: dkardokas
date: 2023-09-12 09:55:00 +0100
categories: [Power Apps]
tags: [Canvas Apps, Model-driven Apps, SharePoint]
---

Embedding an iframe in a Canvas app is sadly not supported natively. To get around this limitation we need to use a PCF control. There are some readily [available](https://github.com/yashag2255/iframePCF) (Thanks @yashag2255) - although it's worth pointing out it is not complicated to build your own, just have to follow the right [steps](https://learn.microsoft.com/en-us/power-apps/developer/component-framework/implementing-controls-using-typescript?tabs=before).

The interesting part starts when you start providing the URL/source for the iframe - a likely scenario is that would be a SharePoint page or a document. All looks good and well in Power Apps Studio, the preview works, then you publish the app and it breaks - displaying a message along the lines of `yourtenant.sharepoint.com refused to connect`. In the console you find a longer version of the message:

>Refused to frame 'https://yourtenant.sharepoint.com/' because an ancestor violates the following Content Security Policy directive: "frame-ancestors 'self' teams.microsoft.com *.teams.microsoft.com *.skype.com *.teams.microsoft.us local.teams.office.com teams.microsoftonline.cn *.powerapps.com *.yammer.com *.officeapps.live.com *.office.com *.stream.azure-test.net *.microsoftstream.com *.dynamics.com *.microsoft.com onedrive.live.com *.onedrive.live.com securebroker.sharepointonline.com".

This is quite strange, because the list of allowed ancestors includes *.powerapps.com - so why is it failing to load the iframe? The answer can be found by inspecting the DOM of a published Power App - when navigating up the node tree, you'll find that the whole app itself is wrapped in an iframe, usually with id `fullscreen-app-host` and src attribute pointing to 'https://pa-static-ms.azureedge.net'. So the URL our component is running at isn't *.powerapps.com - it's *.azureedge.net

In case the embedded page is a custom web app you have control of - the problem can be solved by modifying the [CSP header](https://developer.mozilla.org/en-US/docs/Web/HTTP/Headers/Content-Security-Policy/frame-ancestors)

```csharp
app.Use(async (context, next) =>
{
    context.Response.Headers.Add("Content-Security-Policy", "frame-ancestors 'self' *.powerapps.com *.azureedge.net");
    await next();
});
```

However if the page in question is a SharePoint page/document, we have no such luxury. It may appear there is no way to include a SharePoint preview on a Canvas app (all the more infuriating because of the inclusion of *.powerapps.com as 'allowed' frame ancestors) - but fear not, because there is more than one way to get a document preview, and they are not all equal!
The standard approach is to get full URL of the document, e.g. `https://mytenant.sharepoint.com/sites/mysite/Documents/FileABC.docx`, then add "?web=1" (maybe add "action=view") and call it a day. This doesn't work in this scenario - but there's another, more specialized way - **`/_layouts/15/embed.aspx`** page. This takes `UniqueId` parameter - so you'll need to get the GUID of the file in question, but most importantly this page **does not include `Content-Security-Policy` header**, and so can be embedded more easily (user still needs to be authenticated in the browser).

To recap, the full URL to be used should look similar to `https://mytenant.sharepoint.com/sites/mysite/_layouts/15/embed.aspx?UniqueId=document-guid` - and the document will get loaded.

A side note about yashag2255's PCF solution linked above - it includes frame update in `updateView` callback - which keeps the contents up to date, but wreaks havoc if a scrollbar is introduced (re-rendering on every scroll event). Simple solution is to remove code from `updateView`; also the size of the frame is hard-coded, which needs to be adjusted. I may publish a more flexible version and link here if there's appetite for it.

