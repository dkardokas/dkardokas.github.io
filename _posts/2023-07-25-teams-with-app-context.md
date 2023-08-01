---
title: Provisioning Teams with app-only context
author: dkardokas
date: 2023-07-25 07:55:00 +0100
categories: [SharePoint, MS Teams]
tags: [PnP Core, SharePoint, Code]
---

## Not as easy as it seems

When running code in an app-only context (which is common for provisioning tools), creating a Team site (with associated M365 Group) is not necessarily a straight forward task. 

In this example we're using [PnP Core](https://github.com/pnp/pnpcore), but the same ideas/issues apply with vanilla CSOM or using other tools.
Creating a non-Teams site can be done using [`SiteCollectionManager`](https://pnp.github.io/pnpcore/api/PnP.Core.Admin.Model.SharePoint.ISiteCollectionManager.html)
```cs
{
    var siteColMan = adminContext.GetSiteCollectionManager();
    TeamSiteWithoutGroupOptions siteOpts = new TeamSiteWithoutGroupOptions(new Uri(provisioningProperties.Url), provisioningProperties.Title)
    {
        Owner = provisioningProperties.Owner,
        Description = provisioningProperties.Description,
        Language = PnP.Core.Admin.Model.SharePoint.Language.English
    };
    var siteCtx = siteColMan.CreateSiteCollection(siteOpts)
}
```
However when we try to pass in a `TeamSiteOptions` object instead, it gives a nasty error message 
> REST API Exception occurred

The [`ITeamManager`](https://pnp.github.io/pnpcore/api/PnP.Core.Admin.Model.Teams.ITeamManager.html) class looks promising, however we run into the same problem (albeit with a more descriptive error)

> Operation not supported in app-only context

## Graph API to the rescue

There may be several ways to get around this, but they would all include similar ideas.
Rule #1: use Graph API - forget SharePoint context completely, it isn't going to work. Even though PnP Core is supposed to use Graph API (with the `GraphFirst = true` option), it seems to fall back to SharePoint operations in these specific cases. Instead, we construct a Graph client explicitly

```cs
{
public GraphServiceClient GetGraphClient()
        {
            if(_graphClient == null)
            {
                var keyVault = new AzureKeyVault(Environment.GetEnvironmentVariable("AZURE_KEY_VAULT_URI", EnvironmentVariableTarget.Process));

                X509Certificate2 certificate = keyVault.GetCertificate(Environment.GetEnvironmentVariable("AZURE_KEY_VAULT_CERTIFICATE_NAME"));
                var cred = new ClientCertificateCredential(
                    Environment.GetEnvironmentVariable("AZURE_AD_TENANT_ID"),
                    Environment.GetEnvironmentVariable("AZURE_AD_CLIENT_ID"),                     
                    certificate);
                _graphClient = new GraphServiceClient(cred);
            }

            return _graphClient;
        }
```

Then we can issue the long-winded request, passing in the properties required

```cs
            var requestBody = new Microsoft.Graph.Models.Team
            {
                DisplayName = provisioningProperties.Title,
                Description = provisioningProperties.Description,
                Members = new List<ConversationMember>
                    {
                        new AadUserConversationMember
                        {
                            OdataType = "#microsoft.graph.aadUserConversationMember",
                            Roles = new List<string>
                            {
                                "owner",
                            },
                            AdditionalData = new Dictionary<string, object>
                            {
                                {
                                    "user@odata.bind" , "https://graph.microsoft.com/v1.0/users('OWNER-GUID-GOES-HERE')"
                                },
                            },
                        },
                    },
                AdditionalData = new Dictionary<string, object>
                    {
                        {
                            "template@odata.bind" , "https://graph.microsoft.com/v1.0/teamsTemplates('standard')"
                        },
                    },
            };

            var result = graphClient.Teams.PostAsync(requestBody).Result;
```

Although the code isn't very elegant, it gives a good level of control over the exact parameters being sent to Graph API.

## One More Thing..

For Graph API request to work, make sure you have granted correct permissions to your app - specifically `Team.Create`
![permissions](/assets/img/Screenshot 2023-08-01 182755.png)