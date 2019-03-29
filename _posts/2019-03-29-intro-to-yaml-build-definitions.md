---
layout: post
section-type: post
title: Intro to YAML Build Definition (Build as Code)
category: azuredevops
tags: [ 'azuredevops','yaml' ]
---

Azure DevOps introduced YAML (Yet Another Markup Language) to use in build definitions, which has many advantages over using the traditional visual designer. First, let's go over a description of what YAML is, and how it started. 

YAML was designed with humans in mind. It was structured so it can be easily read by humans, describing exactly what it was going to do. It can be interepted by scripting languages and is easy to implement, especially, in this case, Azure DevOps. 

There are two way to start building your definition using YAML, using a built in template Azure DevOps has provided, which is many or by utilizing the "View YAML" feature in the visual designer. When creating a new build definition, Azure DevOps will provided a plethra of template to start with, a shell of your build definition, that you can customize and tailor to your organization. Another handy feature is by clicking on the "View YAML" button above any task, this will show your task in a YAML format with all values exactly how you had it. All you need to do get started creating YAML build definitions is:

<p  align="left">
- Enable the preview feature under your profile picture and clicking on "Preview Features>Enable YAML Pipeline Creation Experience" 
![Alt Text](/../master/img/previewFeatures.png)
- Create a `azure-pipelines.yml` file in the root of your repo, this tells Azure DevOps that you want to use YAML build definitions.  
</p>

Let's take a look at a very basic YAML definition that copies files to the artifact directory and uploads the artifact.

```pool:
  name: Hosted VS2017
name: $(Date:yyyyMMdd)$(Rev:.r)
trigger:
  branches:
    include:
    - master
  paths:
    exclude:
    - azure-pipelines.yml
steps:
- task: ArchiveFiles@2
  displayName: 'Archive $(Build.SourcesDirectory)'
  inputs:
    rootFolderOrFile: '$(Build.SourcesDirectory)'
    includeRootFolder: false

- task: PublishBuildArtifacts@1
  displayName: 'Publish Artifact: drop'
```

<p  align="left">
- pool: - Defines which agent pool to use when build your application.
    - name: - The name of the agent pool. (Azure Hosted VM with Visual Studio 2017)
- name: - Defines the build number format. In this case, the date and revision incremented with each build. (e.g. 20190329.1).
- trigger: - Enables CI on the build definition.
    - branches: - Defines the git branches to include or exclude in the CI trigger. 
    - include: - Include these branches to the CI trigger. 
    - -master - Uses the master branch.
    - paths: -Defines the paths to include or exclude in the CI trigger.
    - exclude: - Exlude these paths from the CI trigger.
    - -azure-pipelines.yml - Exlude the build definition file from triggering a CI build. We exclude this as we do not want a build to kick off if we modify the YAML file.
- steps: - Defines the build steps to take to build the application. This is the "Agent job" in the visual designer.
- -task: - Defines the task name.
- displayName: - Defines the display name that will show up in the task log when the build is running. ($(Build.SourcesDirectory) is the working directory on the build machine, /_work/##/s)
- inputs: - Defines the arguments of the task. (RootFolderorFile and IncludeRootFolder). If there are no arguments provided in the YAML file, it assumes the default usuage in the task. e.g. The Publish Build Artifacts Task defaults the "drop" artifact to $(Build.ArtifactStagingDirectory)
</p>

The example above demostrates the advantages of using the YAML experience over the visual designer.

### Traceability

The build definition sits in the root of the repo. We can now use native git command to see changes made to the build definitions. 

### Visibility 

There is no need to go log into the Azure DevOps site to view the build definition. Open the azure-pipelines.yml file to view the current build definition.

### Easability

Making changes to the build definition is made in your local repo instead of creating a draft build pipeline using the visual designer. 

### Builds as Code

We all know that automation is taking over and building Infrastructure as Code is becoming mainstream. This defines "Builds as Code," automating the of creation build and release definitions. 

At the time of this writing, Azure DevOps only supports YAML in the build definition pipeline. There are plans to introduce YAML release definitions, however, it looks like it is slated for sometime this quarter. Transitioning to YAML build definitions will add value to your organization and should be a stepping stone to automating your build definitions in Azure DevOps.

Microsoft has created a reference on how to take YAML builds a step further, adding variables, pull request triggers, and jobs. As well as providing all of the built in tasks, arguments for each, and examples. You may ask, what about extensions the community has created? How can I find out how to use extensions? Github is your friend. Most extensions created by the community have their extension accessible in GitHub, discover the tasks, arguments, and how to use them. Another option is to create the task in the visual designer and view the yaml from the task. 

pwshliquori

## Resources

### YAML Schema

https://docs.microsoft.com/en-us/azure/devops/pipelines/yaml-schema?view=azure-devops&tabs=schema

### Learn YAML

https://learnxinyminutes.com/docs/yaml/

### Builds as Code Lab

https://www.azuredevopslabs.com/labs/azuredevops/yaml/