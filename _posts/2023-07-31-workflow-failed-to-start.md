---
title: Connection not configured for this service
author: dkardokas
date: 2023-07-31 07:55:00 +0100
categories: [Dataverse, Bugfix]
tags: [Power Platform, Dataverse, Dynamics]
pin: true
---

## What happened?

Several clients have been reporting an issue where a Canvas App is unable to run an (on-demand) Flow, throwing an error "The workflow has failed to start. Connection not configured for this service" error
![error](/assets/img/112331.png)

## What is causing this?

As of this writing, this is actually expected behaviour in a non-default Power Platform environment. Basic User role does not provide access to view or run Flows created by other users. You can inspect this by navigating to your environment settings -> Security Roles -> Basic User and find entity Process (workflow):
![basic user role](/assets/img/Screenshot 2023-07-31 113111.png)

In addition, you will also see a message 'The Basic User role privileges cannot be adjusted due to its non-customizable nature' at the top. While there are [suggestions](https://powerusers.microsoft.com/t5/Using-Connectors/connection-not-configured-for-this-service/m-p/2269002){:target="_blank"} to update permissions for Basic User, this is no longer allowed by Microsoft.

## Fixing the issue

To resolve the issue, a new custom role is required. Create one from Security roles -> New role. Give it a name and select Business unit (if unsure, default organization Business unit should be fine). Member's privilege inheritance can be left as default (Direct User (Basic) access level and Team privileges). After saving, the default value for Process (workflow) entity Read access should already be set to Organization (which is what we want) - if not, change it to Organization value.
![custom role](/assets/img/Screenshot 2023-07-31 115134.png)

The last part is to assigned the new role to your users. In test/dev environment it should be fine to assign directly, however in Prod the best way is to do it through a Team. If there is already an AAD group encompassing all users in the organization (or at least those likely to user this Power App/environment), create a new Team (Environment settings-> Teams -> Create team) and choose type AAD Security Group (or Office Group if you have an M365 group). Fill in required values and save, then select Manage security roles and assign the newly created role.

Flows should now be running as expected. Issues? drop a comment below.


