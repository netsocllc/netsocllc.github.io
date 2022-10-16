# Detecting Privilege Escalation via High Privileged Role Assignments


# Introduction

It's no secret that information security teams should have detections in place to detect unusual changes in high privilege roles. Accidentally assigning the wrong Azure AD role can be catastrophic if a high privileged user or managed identity is to unknowingly fall victim to compromise. There are a number of roles which should be considered sensitive, these should be heavily monitored and periodically audited.

Some examples of these roles include but are not limited to the following:

*Global Administrator*

*Company Administrator*

*Privileged Authentication Administrator*

*Privileged Role Administrator*

![image-20221016183058877](/posts/Detecting-High-Privileged-Role-Assignments.assets/image-20221016183058877.png)

Note: As if having to monitor and detect changes in permissions with our users wasn't a complex task, organizations should equally monitor the permissions and roles across APIs more on this here: https://blog.netsoc.us/detecting-privilege-escalation-via-azure-api-permission-abuse/

## Requirements

In order to detect changes to high privileged roles in Azure AD, it's required the data connector "Azure Active Directory" be connected and Audit Logs be ingested into Azure Sentinel. Global Administrator or Security Administrator is required in order to make this connection, along with the proper permissions at the Workspace or Resource group level to make changes to the workspace. Additionally, read and write permissions to AAD diagnostic settings must be available.

![image-20221016183203663](/posts/Detecting-High-Privileged-Role-Assignments.assets/image-20221016183203663.png)

## KQL 

Query to detect changes across the aforementioned roles, feel free to add additional high privileged roles like Security Administrator, Exchange Administrator, or Application Administrator.

`let HighPrivRoles = dynamic(["Global Administrator","Company Administrator","Privileged Authentication Administrator","Privileged Role Administrator"]);`
`AuditLogs`
`| where OperationName == "Add member to role"`
`| mv-expand TargetResources`
`| mv-expand TargetResources.modifiedProperties`
`| where TargetResources_modifiedProperties.displayName == "Role.DisplayName"`
`| extend AddedToRole = replace_string(tostring(TargetResources_modifiedProperties.newValue),'"','')`
`| where AddedToRole in~ (HighPrivRoles)`
`| extend Actor = iff(isnotempty(InitiatedBy.user.userPrincipalName),InitiatedBy.user.userPrincipalName,InitiatedBy.app.servicePrincipalId)`
`| extend TargetUsername = TargetResources.userPrincipalName`

Depending on your environment, it may be beneficial to monitor specific changes like password changes across these high privileged roles. Again, feel free to add additional roles or activity as necessary.

`let HighPrivRoles = dynamic(["Global Administrator", "Company Administrator", "Privileged Authentication Administrator", "Privileged Role Administrator"]);`
`AuditLogs`
`| where OperationName == "Reset user password"`
`| mv-expand TargetResources`
`| extend TargetUsername = tostring(TargetResources.userPrincipalName)`
`| join kind=innerunique (`
    `IdentityInfo` 
    `| where TimeGenerated > ago(14d)`
    `)`
    `on $left.TargetUsername == $right.AccountUPN`
`| mv-expand AssignedRoles`
`| extend AssignedRoles = tostring(AssignedRoles)`
`| where AssignedRoles in (HighPrivRoles)`
`| summarize by TimeGenerated, TargetUsername, AssignedRoles, OperationName, AADUserId=AccountObjectId`

Pro tip: It may be a good idea to generate a weekly or bi-weekly report to audit and review these changes.

