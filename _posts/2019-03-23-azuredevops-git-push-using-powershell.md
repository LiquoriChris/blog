---
layout: post
section-type: post
title: Azure DevOps - Git push using PowerShell
category: azuredevops
tags: [ 'azuredevops','powershell' ]
---

Pushing code to Azure DevOps using GitTfs repos is an easy process, if you know git commands, however, tools such as Visual Studio Code and Visual Studio have source control plugins built into the application. But, what if you wanted to push to a repo during the release pipeline? How would you do it? One way, is to use git commands as a cmd task to `git push` any changes from the previous task, or you can use Azure DevOps' Rest Api and PowerShell to push to a repo. IMO, the second method would be best as you can use the built in variables and use the System.AccessToken variable. In the last post, we now know how to enable the OAuth token for scripts using the Rest API and PowerShell, let's take it a step further.

First, we need to figure out what we are actually pushing to the repo, let's move some files to Archive after a production release. And let's use variables as much as possible to make the command more dynamic. 

```
$Params = @{
    Uri = "$(System.TeamFoundationCollectionUri)$(System.ProjectName)/_apis/git/repositories/$(Build.Repository.Id)/pushes?api-version=5.0"
    Headers = @{
        Authorization = "Bearer $(System.AccessToken)"
    }
    Body = $here
    ContentType = 'application/json'
    Method = 'Post'
    ErrorAction = 'Stop'
}
Invoke-RestMethod @Params
```

But what about the actual changes, let's break down the body and show how to move files to another directory. Using the here-string makes it possible to use JSON directly instead of converting a PowerShell hash table to JSON. 

```
$here = @"
{
    "refUpdates": 
    [
        {
            "name": "$(Build.SourceBranch)",
            "oldObjectId": "$(Build.SourceVersion)"
       }
    ],
       "commits": 
        [
            { 
               "comment": "Moving scripts to Archive",
               "changes": [
                   {`
                       "changeType": "rename",
                       "sourceServerItem": "/current/sqlScripts",
                       "item": {
                           "path": "/archive/test/$(Build.BuildNumber)"
                        }
                    }
                ]
            }
        ]
    }
"@
```

`$(Build.SourceBranch)` is the branch that is being using to push the code. e.g. refs/heads/master  
`$(Build.SourceVersion)` is the lastest commit id from the repo  
`comment` is the commit comment. Same as using `git commit -m 'pushing'`  
`changeType` is what is being done to the repo, in this case `rename` which is also used for move  
`sourceServerItem` is the source file or folder that is being moved  
`path` is the destination folder to move the file or folder  
`$(Build.BuildNumber)` is the build number that was generated from the build pipeline

This example will move the current sql scripts to an archive folder with the build number as the folder name, creating a relationship between the build number and what scripts were ran at release time. Let's look at another example where we create folders after moving the sql scripts to archive.

```
$Params = @{
    Uri = "$(System.TeamFoundationCollectionUri)$(System.ProjectName)/_apis/git/repositories/$(Build.Repository.Id)/pushes?api-version=5.0"
    Headers = @{
        Authorization = "Bearer $(System.AccessToken)"
    }
    Body = $here
    ContentType = 'application/json'
    Method = 'Post'
    ErrorAction = 'Stop'
}
Invoke-RestMethod @Params
``````
```
$here = @"
{
    "refUpdates": [
        {
            "name": "$(Build.SourceBranch)",
            "oldObjectId": "$(Build.SourceVersion)"
        }
    ],
    "commits": [
        {
            "comment": "Moving scripts to Archive",
            "changes": [
                {
                    "changeType": "add",
                    "item": {
                        "path": "/current/sqlscripts/1.sql"
                    },
                    "newContent": {
                        "content": "# Auto generated scripts. Do not delete.\n\nTBD",
                        "contentType": "rawtext"
                    }
                }
           ]
        }
    ]
}
"@
```

In the above example, only a couple properties have changed.

`changeType` is what is being done to the repo, in this case `add` adding folders to the repo  
`newContent` content to be added into the file being created

Other changes you can make in a repo are:  
<p  align="left">
- Delete <BR>
- Initial Commit <BR>
- Rename <BR>
- Update <BR>
</p>

Since you cannot add an empty folder, a placeholder file needs to be created if you are adding folders into the repo. The two examples demostrate how awesome using PowerShell automation really is, taking a manual process of moving files to archive and completely automating the process. Adding value to your organization by automating processes that were being done manually really stands out and can help promoting yourself and your career. 