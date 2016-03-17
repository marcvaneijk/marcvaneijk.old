---
layout: post
date: 2016-03-18
author: Marc van Eijk
title: Azure Resource Manager - Organizing your templates
tags: Azure Resource Manager, structure, templates, readme, markdown, workflow
published: false
---

After you have mastered the basics of template orchestration in Azure Resource Manager, you find templates and samples in multiple locations. You might have some copies on your local disk, a couple of Visual Studio project containing .json deployment templates, you might have created a GitHub account and stored some templates in a repository. This blog will help with organizing your template, by standardizing your workflow. 

## Prerequisites

	- GitHub account
	- Git or GitHub Desktop
	- Visual Studio with the latest Azure SDK

Check [Azure Resource Manager – Getting started with GitHub](/2016/02/03/githubstart) for a step by step to create a GitHub account and install Git or GitHub Desktop.

## Folder structure

Start by creating a repository in your GitHub account and clone the repository, so you have a local copy. Decide on a folder structure for your deployment template scenarios. Each deployment template and related files should be contained in its own folder. Give the folder a descriptive name, that allows you to easily identify what the kind of deployment template scenario the folder contains. Since some URLs in the templates can reference the path of the template that contains the folder names, it is a good idea to keep the folder names short and preferably without spaces.

Depending on the amount of templates, you can set up a different folder structure. If you do not have a lot of deployment template scenarios, you can create a folder for each deployment template scenario directly in the root.

```
\GitHubRepository
	\SimpleVM
	\SharePointFarm
	\SqlWebApp
	\StorageNetwork
```

If you have a lot of deployment template scenarios it can make sense to add one level of folders.

```
\GitHubRepository
	\IAAS
		\SimpleVM
		\ShrePointFarm
		\StorageNetwork
	\PAAS
		\SqlWebApp
```

No matter what folder structure you come up with. Think through the pros and cons and don't make it too complex.

## Files and naming convention

It help to standardize the content of each deployment template scenario. You can standardize by using a standard file set and consequent naming convention. Each deployment template scenario should contain the following files.

- README.md
- azuredeploy.json
- azuredeploy.parameters.json

Specify the values for the parameters in the azuredeploy.parameters.json file, according to rules defined in the template (allowed values etc.). Depending on the assets used, subfolders help to organize your assets. The custom scripts that are needed for successful template execution should be placed in a folder called scripts. Linked templates should be placed in a folder called nested. Images used in the README.md should be placed in a folder called images. 

- scripts
- nested
- images


GitHub uses ASCII for ordering files and folder. For consistent ordering create all files and folder in lowercase. The only exception to this guideline is the README.md, that should be in the format UPPERCASE.lowercase.

**README.md**

A readme is very important for a deployment template scenario. It is usually overlooked or contains just a couple of lines of text. The README.md describes your deployment. A good description helps other community members to understand your deployment. The README.md uses GitHub Flavored Markdown for formatting text. If you want to add images to your README.md file, store the images in the images folder. Reference the images in the README.md with a relative path (e.g.  ![alt text](images/namingConvention.png "Files, folders and naming conventions") ). This ensures the link will reference the target repository if the source repository is forked. A good README.md contains the following sections.

- Deploy to Azure button
- Description of what the template will deploy
- Tags, that can be used for search. Specify the tags comma separated and enclosed between two back-ticks (e.g  Tags: cluster, ha, sql )
- *Optional: Prerequisites
- *Optional: Description on how to use the application
- *Optional: Notes

> Do not include the parameters or the variables of the deployment script

Instead of describing all the individual pieces of the README.md file, you can find a sample README.md here. 

## Starter folder

Instead of creating a deployment template scenario folder with all files from scratch it is much easier to copy the complete folder with all files from a starter folder. Create a starter folder in the root of your local clone. Create an blank deployment template in Visual Studio (you can even decide to add an [object variable](/2016/03/14/nested) with the template link).

You can copy and store the sample README.md file in your own repository and copy that to each new deployment that you create. You only need to update it to reflect the scenario and you done. All README.md will have a standard layout this way, without requiring a lot of work from your end.

Make sure you also create a README.md in the root folder of your repository describing the content and preferably the contribution guidance if people want to contribute to your code.

## Workflow

Now that you have defined your folder structure in your local clone, push the changes to your repository in GitHub. You can find the steps to complete this procedure in [Azure Resource Manager – Getting started with GitHub](/2016/02/03/githubstart). 

To create a new deployment template scenario using the following workflow.

- Copy the starter template and rename the copied folder describing the scenario
- Update the README.md in the new folder
- Create a new Azure Resource Group template and give it the same name as scenario folder



- Remove the azuredeploy.json template from the project
- Add the azuredeploy.json from your deployment folder by right clicking the template folder in the solution explorer, select add, existing item.



- Work your template magic in Visual Studio
- Test the template by directly deploying it from Visual Studio



- Save the template
- Push the changes to your repository in GitHub
