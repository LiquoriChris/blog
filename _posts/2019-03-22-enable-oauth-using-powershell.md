---
layout: post
section-type: post
title: Azure DevOps - Enable OAuth Token using PowerShell
category: azuredevops
tags: [ 'azuredevops','powershell' ]
---

Azure DevOps allows us to run custom scripts to help our software and infrastructure get delivered quickly. There are times that the scripts run without an issues, however, sometimes there is a need to invoke the Azure DevOps Rest API in the CD pipeline. Sure, you can create a script using the API, authenticating with Azure DevOps with a personal access token and should work, but there is a better solution.

Allowing scripts to access the oauth token authenticates the script with the System.AccessToken variable, which runs as the Project Collection Build Service, a built-in service account in Azure DevOps. Today, we will be taking a look on how to enable this feature using PowerShell. 

Since the feature needs to be enabled per release definition, the first item we need to find is the ID of the release definition. This can be found by using the Rest API or in the URL when clicking on the release definition in Azure DevOps. Since we are using PowerShell, let's try it, but first, be sure to have your personal access token handy.

```
$Params = @{
    Uri = "https://dev.azure.com/pwshliquori-blog/blog/_apis/release/definitions/1?api-version=5.0"
    Headers = @{
        Authorization = "Basic $ConvertToBase64"
    }
}
$Def = Invoke-RestMethod @Params
```

Let's take a look at the command:
```
$Params: A hash table we will be splatting later on when we are ready to run the command.
```
```
$Params.Uri: The components needed to get the release definitions.  
pwshliquori-blog: Organization name.  
blog: Project name.  
_apis: Calling the rest api.  
release: The area of the api call.  
definitions: The resource of the api call.  
api-version=5.0: The latest version of the api. 
```
```
$Headers: Authorization header using your base 64 encoded personal access token.
```
```
Invoke-RestMethod @Params: Invokes the Rest API splatting the $Params variable.
```

The command should return all release definitions in the project. Now we need to dig down and find the property needed to enable, in this case: "enableAccessToken"

The enableAccessToken property is set to false by default, lets find and set it to true:

```
$Def.environments.deployPhases.deploymentInput
$Def.environments.deployPhases.deploymentInput.enableAccessToken = $true
$Def.environments.deployPhases.deploymentInput
```

Now that we set the "enableAccessToken" to true, we need to update the release definition with the changed value. To do this, we need to convert the $Def variable to JSON format and set the ContentType to application/json.

```
$Body = ConvertTo-Json -InputObject $Def -Depth 100

$Params = @{
    Uri = "https://dev.azure.com/pwshliquori-blog/blog/_apis/release/definitions/1?api-version=5.0"
    Headers = @{
        Authorization = "Basic $ConvertToBase64"
    }
    Body = $Body
    ContentType = 'application/json
    Method = 'Put'
}
Invoke-RestMethod @Params
```

The body needs to contain the entire release definition with the updated "enableAccessToken" property. After running the command, we can now utilize the System.AccessToken to run scripts and processes using OAuth authentication against the Project Collection Build Service account. By using PowerShell, we can now turn the commands above into a function to automate the process of enabling this feature. 

In the next blog post, we will take a look at using PowerShell and Azure DevOps Rest API to push a commit in a GitTfs repository.