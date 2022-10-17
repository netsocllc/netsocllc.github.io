---
title: "Secure and Monitor Cross-Tenant Privileged Access"
date: 2022-10-17T05:15:03Z
draft: false
---

## Introduction

The underlying technology surrounding cross tenant administration is incredibly fascinating. Managed Service Providers often leverage Azure Lighthouse or Foreign Principle Access to delegate permissions. While this feature allows for some luxurious integrations, they may not be aware they are introducing more risk to themselves and their customers. In this article we will discuss how to securely delegate, monitor, and harden privileged access.

Note: This article applies to all organizations leveraging cross-tenant technology and is not specific to Managed Service Providers. This article entails a way to increase security cross-tenant delegation of permissions and resources.

## Overview

At the end of this article you will understand how to harden and remove permanent access delegations and replace it with a more secure temporary session based elevation of privilege to specific customer environments.

![image-20221017141543883](/posts/Securing-and-Monitoring-cross-tenant-privileged-access-and-administration.assets/image-20221017141543883.png)

## Removing Permanent Access

This is done by leveraging Privileged Identity Management's new feature which allows for the temporary session based elevation of group memberships. Often times Managed Service Providers leverage one group to delegate permissions into customer environments either via Foreign Principle Access or Azure Lighthouse. This access control model without being subject to periodic audits can create headaches for Managed Service Providers when trying to restrict or control the access of their employees into customer environments. Additionally, assigning a user to a group gives access the access required until the group membership is removed. With Azure AD Privileged Identity Management, the user can temporarily activate their group membership for up to 8 hours. It is also recommended the managing tenant create a separate group for each customer environment, role, or both.

Assigning the group to a user via Privileged Access Groups will require the delegation of access to be controlled by Azure Active Directory Privileged Identity Management. To do this, create a group and enable it for "Privileged Access" under group settings and add the users who require access into customer environments. Following this implementation, those users who will require access will first need to activate their group membership in the Azure Management Portal under Privilged Identity Management.

![image-20221017145405253](/posts/Securing-and-Monitoring-cross-tenant-privileged-access-and-administration.assets/image-20221017145405253.png)

Note: It is never a bad idea to require MFA verifications

## Tenant Access Delegation Hardening

It is highly recommended that this is done for all subscriptions across managing and customer tenants. In Azure Policy, there is an available built-in definition called "Allow managing tenant ids to onboarding through Azure Lighthouse." This built-in definition will restrict Azure Lighthouse delegations to specific managing tenants in a "deny-all" fashion to increase security by limiting those who can manage Azure Resources. With this policy, you are supposed to list the tenants who are allowed to be delegated as service providers, it is required you assign this policy at the subscription level. 

![image-20221017150551438](/posts/Securing-and-Monitoring-cross-tenant-privileged-access-and-administration.assets/image-20221017150551438.png) 

## Effective Auditing Surrounding Cross Tenant Administration

In the diagram at the top of this article, you'll notice that the Azure Activity logs are being sent to the same workspace. While this isn't technically possible, there are very interesting deployment models surrounding the use of Azure Lighthouse and Microsoft Sentinel features allowing the managing tenant to query across multiple workspaces (more on this in another blog post). Regardless it is highly recommended that the following is monitored across all tenants and subscriptions which delegate access to managing or customer tenants:

1. The use of MFA when a user from another tenant authenticates
2. When a new user from the managing tenant becomes eligible for Privileged Access Group assignments
3. When an eligible user activates the Privileged Access Group assignment
4. When a user from the managing tenant makes changes in the customer tenant
5. When resource delegations are assigned to any tenant other than the managing or customer tenant

### KQL Queries

Queries for these monitoring detections can be found in previous blog posts:

https://blog.netsoc.us/detecting-high-privileged-role-assignments/

https://blog.netsoc.us/detecting-privilege-escalation-via-azure-api-permission-abuse/

https://blog.netsoc.us/detecting-azure-lighthouse-attacks/

