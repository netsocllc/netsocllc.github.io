# Deploy Custom Content In Microsoft Sentinel Using Repositories


# Introduction

It can be stressful managing multiple workspaces in Microsoft Sentinel, especially when it comes to having to manually deploy analytic rules in large enterprise environments with multiple tenants. This feature is incredibly useful for Managed Security Service Providers who manage multiple workspaces. With Microsoft Sentinel Repositories, we can deploy and manage custom content from a central repository across multiple workspaces at ease. 

Originally this article was written in early 2022 when Microsoft Sentinel Repositories was released. This feature is still in public preview and can be referenced here: https://learn.microsoft.com/en-us/azure/sentinel/ci-cd?tabs=github

# Prerequisites

Sentinel supports connections to GitHub and Azure DevOps repositories. According to Microsoft the following permissions are required to connect a Microsoft Sentinel workspace to your source control repository:

- Resource Group Owner or both the User Access Administrator and Sentinel Contributer roles
- Contributer access to the code repository in GitHub or Azure DevOps
- Actions enabled for GitHub and Pipeliens enabled for Azure Devops

![image-20220927081028648](/Users/bfell/Desktop/blog/netsocllc.github.io/content/posts/Deploy-Custom-Content-In-Microsoft-Sentinel-Using-Repositories.assets/image-20220927081028648.png)

Pro tip: For the MSSPs and multi-tenant scenarios leveraging Azure Lighthouse, it's recommended assigning these roles to a group in your ARM template prior to deploying Azure Lighthouse. The role definition ids can be found here: https://learn.microsoft.com/en-us/azure/role-based-access-control/built-in-roles

## Additional Prerequisites

Additionally, to deploy custom content it's required to have Actions enabled for GitHub and Pipelines enabled for Azure DevOps. Although there is a plethora of .yml files in the Microsoft Sentinel Public GitHub Repository, custom content can only be deployed via Azure Resource Manager (ARM) templates. 

Pro tip: It is extremely useful to leverage a separate workspace as a model tenant to build, test, stage, and export custom content as ARM templates in Microsoft Sentinel.

![Untitled picture2](/Users/bfell/Desktop/blog/netsocllc.github.io/content/posts/Deploy-Custom-Content-In-Microsoft-Sentinel-Using-Repositories.assets/Untitled picture2.png)



# Deploying Content

For the purpose of this demonstration we'll be deploying Analytic Rules, but the following content types can be selected when creating a new connection in Repositories:

- Analytic Rules
- Automation Rules
- Hunting Queries
- Parsers
- Playbooks
- Workbooks

## Creating Connections

At the time of writing there were too many cross-tenant limitations which existed when connecting Azure DevOps so this demonstration was done using GitHub. If you decide to use Azure DevOps, please note that if you're creating the connection as a guest user the AzureDevOps URL must be entered manually and will not appear in the dropdown.

Be sure to be signed into either GitHub or Azure DevOps to connect a repository and in Microsoft Sentinel, under Content Management, select Repositories in the Sentinel workspace where you wish to deploy content.

![image-20220927080432943](/Users/bfell/Desktop/blog/netsocllc.github.io/content/posts/Deploy-Custom-Content-In-Microsoft-Sentinel-Using-Repositories.assets/image-20220927080432943.png)

Make sure you're signed into the account which has been assigned the proper permissions, select "Add new", and Create a new connection.

Name the connection and if necessary provide a description, next authorize your Source control application. Once you've authorized either GitHub or AzureDevOps select a repository, branch, and select the Content types and hit create.

![image-20220927082536123](/posts/Deploy-Custom-Content-In-Microsoft-Sentinel-Using-Repositories.assets/image-20220927082536123.png)

This connection will create a workflow rule in your source repository and begin to deploy or update any existing content specified in the rule. For the sake of this article, we're going to add the rule after we connect the repository.

## Example

In the example below, we'll deploy a really simple analytic rule in which detects for failed login activity in an Active Directory network.

![Untitled picture](/Users/bfell/Desktop/blog/netsocllc.github.io/content/posts/Deploy-Custom-Content-In-Microsoft-Sentinel-Using-Repositories.assets/Untitled picture.png)

Since we can only deploy content through Azure Resource Manager (ARM) templates in the form of .json files, it is recommended to first create the rule in a separate workspace and export it. You can do this by selecting the analytic rule and selecting the export button at the top.

Once you have the .json file ARM template, adding this rule to your source repository will immediately deploy to all of the connected workspaces.

![image-20220927084908815](/Users/bfell/Desktop/blog/netsocllc.github.io/content/posts/Deploy-Custom-Content-In-Microsoft-Sentinel-Using-Repositories.assets/image-20220927084908815.png)

Note that any changes to existing content will overwrite, any content matching the parameters to deploy will deploy to all workspaces connected to the workflow.

![image-20220927085306366](/Users/bfell/Desktop/blog/netsocllc.github.io/content/posts/Deploy-Custom-Content-In-Microsoft-Sentinel-Using-Repositories.assets/image-20220927085306366.png)

Pro tip: Making use of a naming convention to label your CICD content can be extremely helpful when managing content. When managing large rule sets with multiple workspaces, it's recommended making the required updates and changes to rules from the central repository to maintain consistency across the connected workspaces.
