---
title: "Detecting Privilege Escalation via Azure API Permission Abuse"
date: 2022-10-16T05:15:03Z
draft: false

---

# Introduction

Effective audit trails surrounding sensitive groups is essential when it comes to maintaining secure active directory networks and in some cases required by regulatory compliance frameworks. For cloud and hybrid-AD joined networks there is twice as much work to be done. Leveraging automation to have detections and reporting in place for security professionals is considered best practice and is critical in terms of detecting privilege escalation attacks both on premise and in the cloud. Unfortunately the monitoring of user privileges is no longer enough as more on premise networks connect to the cloud, thus adding more complexity and increasing the threat landscape. 

## Detecting Azure API Permission Abuse

Ensuring we grant the right permissions to enterprise applications and application registrations is equally important as doing the same with our users. In some cases, fundamental application management can mitigate more risk than we would expect considering powerful roles and attack paths surrounding APIs. Depending on the initial access of a threat actor, one could create a back door by creating a custom app registration and assigning one of many highly privileged roles available. This back door can go unnoticed without the proper detections in place.

### Roles

Thankfully there is a built-in alert with Microsoft Cloud App Security "Unusual addition of credentials to an OAuth app that can be used as an indicator of malicious activity. Use the below query to detect these dangerous permissions being added to an application: 

*Application.ReadWrite.All*

*AppRoleAssignment.ReadWrite.All*

*RoleManagement.ReadWrite.Directory*

Information on what permissions are granted through these roles can be found here: https://learn.microsoft.com/en-us/graph/permissions-reference

###  KQL

**let DangerousPermissions = dynamic(["AppRoleAssignment.ReadWrite.All","Application.ReadWrite.All","RoleManagement.ReadWrite.Directory"]);*
 *AuditLogs*
 *| where OperationName == "Add app role assignment to service principal"*
 *| where Result =~ "success"*
 *| mv-expand TargetResources*
 *| mv-expand TargetResources.modifiedProperties*
 *| where TargetResources_modifiedProperties.displayName == "AppRole.Value"*
 *| extend InitiatingUserOrApp = tostring(InitiatedBy.user.userPrincipalName)*
 *| extend InitiatingIpAddress = tostring(InitiatedBy.user.ipAddress)*
 *| extend UserAgent = iff(AdditionalDetails[0].key == "User-Agent",tostring(AdditionalDetails[0].value),"")*
 *| extend AddedPermission = replace_string(tostring(TargetResources_modifiedProperties.newValue),'"','')*
 *| where AddedPermission in~ ( DangerousPermissions )*
 *| mv-expand TargetResources.modifiedProperties*
 *| where TargetResources_modifiedProperties.displayName == "ServicePrincipal.ObjectID"*
 *| extend ServicePrincipalObjectID = replace_string(tostring(TargetResources_modifiedProperties.newValue),'"','')*
 | extend timestamp = TimeGenerated, AccountCustomEntity = InitiatingUserOrApp, IPCustomEntity = InitiatingIpAddress*

 
