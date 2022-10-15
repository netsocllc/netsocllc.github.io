---
title: "Detecting Azure Lighthouse Attacks"
date: 2022-10-15T10:15:03Z
draft: false
---

# Introduction

*Note: This article assumes the reader has 100-level understanding of how to manage resources across tenants through the use of Azure Lighthouse.*

Azure Lighthouse is extremely useful delegating permissions and resources across large multi-tenant enterprise cloud environments or for Managed Service Providers to manage their customer environments. Unfortunately Azure Lighthouse can also create security risks if not monitored properly. Below we will discuss how to detect changes in permissions/authorizations across users, groups, or service principles in cross-tenant scenarios.

More information on Azure Lighthouse here: https://azure.microsoft.com/en-us/products/azure-lighthouse/#overview

## Attack

An attacker can craft an ARM template which can be used to delegate permissions inside the victim's tenant. The threat actor can assign these delegated permissions at the subscription or resource group level and trick an administrator into deploying this template or do it themselves using a hijacked account. A separate is deployment is necessary for each different subscription so if an attacker was to delegate permissions to more than just one, multiple alerts would fire in this scenario. An attacker can also leverage Azure Policy to deploy with automation.

Once lighthouse is deployed an attacker can access the victim's target tenant resources from their own tenant. This is very dangerous considering the contributer role can be delegated using Azure Lighthouse. In order to see current delegations you must check which service offers are currently active in the Auzre Portal under "Service Providers." 

![image-20221015103432899](/posts/Detecting Azure Lighthouse Attacks.assets/image-20221015103432899.png)

## Detection

It is extremely important for us as an MDR provider to detect the latest attack paths both on premise and in the cloud. We dedicate ourselves to extensive research across the latest attack-paths which can be leveraged to take over cloud or hybrid-AD joined active directory networks.

Below you'll find the KQL query which can be leveraged to hunt for suspicious granting of resources via Microsoft.ManagedServices registrations as well as how to detect any operation done by a user from another tenant.

Registration:

`AzureActivity
| where OperationNameValue =~ "Microsoft.ManagedServices/registrationAssignments/Write"
| extend timestamp = TimeGenerated, AccountCustomEntity = Caller, IPCustomEntity = CallerIpAddress`

Activity:

`let HomeTenantId = "YOURTENANTID";
AzureActivity
| extend TenantId = todynamic(Claims).['http://schemas.microsoft.com/identity/claims/tenantid']
| where TenantId != HomeTenantId
| where isnotempty( TenantId )
| sort by TimeGenerated`

