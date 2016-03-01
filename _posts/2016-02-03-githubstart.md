---
layout: post
date: 2016-02-03
author: Marc van Eijk
title: Azure Resource Manager – Getting started with GitHub
tags:
---
You might be totally new to GitHub, maybe you have already worked with it a little or you might even consider yourself a more advanced user and work with GitHub frequently. If you have looked at GitHub before you might have noticed that Microsoft is increasingly using it for multiple purposes. The documentation for Microsoft Azure is hosted on GitHub. This means that you are able to contribute to that documentation (in a controlled fashion of course). Besides documentation, you will also find a lot of GitHub repositories that contain code for interacting with different Microsoft products. A very good example of this is the azure-quickstart-templates repository.

<https://github.com/Azure/azure-quickstart-templates>

This repository contains over 250 different deployment templates to use with Azure Resource Manager.

<img src="/images/2016-02-03/Awesome.png" width="700">

If a deployment template in this repository also complies with certain requirements, they are even displayed on a gallery on the public Microsoft Azure website.

<https://azure.microsoft.com/en-us/documentation/templates/>

In this blog we’ll go step by step, to get you started with contributing on GitHub. I have worked my way through many blogs, finding a piece of the puzzle in each. This blog is long, but if you just walk through it step by step, you can contribute to your own repositories, other repositories and even the azure-quickstart-templates repository. So you can have your own template (with your name, and photo) shining on the public Microsoft Azure website for everybody to deploy to their Microsoft Azure subscription.

## Azure Resource Manager

Azure Resource Manager was announced at Build in April, 2014. This introduced a series of major improvements to Microsoft Azure. It fundamentally changed the platform and enabled a lot of new capabilities which empowered organizations of any type and size to use the enterprise grade platform for all their needs. Azure Resource Manager (ARM for short) enforced a common contract for all resource providers, which allows you to reuse your tooling, scripts and knowledge for any type of resource. Azure Resource Manager also introduced features like RBAC, Template Orchestration, Tagging, Auditing, Resource group and resource locks. It’s a couple of words in a single sentence but these capabilities make a world of difference in the platform.

## Template orchestration

Azure Resource Manager uses templates to orchestrate the deployment of resources in Microsoft Azure and Microsoft Azure Stack. These templates are configured in JSON (JavaScript Object Notation) language. The templates are declarative and idempotent.

- Declarative. You define the end state and the dependencies of the resources that comprise your application in the template. The platform converts the declarative template to imperative code that deploys your resources
- Idempotent. After you deployed your resources with a template and make an update to it, you can rerun the updated template to the same resource group. The updated resources will be added by the deployment.

Besides the deployment of the resources Microsoft Azure and Microsoft Azure Stack provide you with consistent VM extensions. A VM extension allows you to pass scripts to the Operating System. A great example of this is the PowerShell DSC VM extension. PowerShell DSC is also Declarative and idempotent. The VM extension is a resource in a JSON template and you can create dependencies on these resources as well. So if you want to deploy a complete multitier application (for instance a SharePoint farm), you can deploy that as one integral process. This forms a solid foundation for IT PROs and DEVs to bundle their knowledge and drastically improve the agility of their efforts.

## Collaboration

When I started working with ARM Templates over a year ago, I quickly understood the basic template language. Some simple templates, like creating a storageAccount or a virtualNetwork, helped with that understanding. I stored the scripts on my OneDrive, so I could easily access them from anywhere. Really great for myself, but then I had some friends from the community looking for some samples. Sending them an email with my script, resulted in two copies of the same script. They were editing their own version and so was I. That wasn’t a really good experience. By that time Microsoft published a set of quickstart templates on GitHub. The first reaction I had was GitHub = Developer. I was able to go the URI of the quickstart templates open a template and copy/paste in my own environment for further editing. A great location to look at these samples, but I didn’t have a clue how to contribute to these templates. I did see other people adding their templates and contributing to other templates.
<!--more-->

## What is GitHub

Before you get started with GitHub is important to understand what it is and why you should use it. To understand GitHub you should first understand what Git is. Git is a Version Control System (VCS) records changes to a file or set of files over time so that you can recall specific versions later. Git is a local command line application that allows you to work locally (on your machine) with multiple versions of one ore multiple files. With the ARM templates in mind, Git can add versioning to the edits you are making to the local template over time. Which is great but still does not allow you to collaborate with other people. Your files are stored locally on your machine. This is where GitHub comes in.

GitHub is a web-based Git repository hosting service. I’ll try to simplify with an example. Instead of storing the folder with your ARM templates locally on your machine, you can store the templates in a folder on GitHub. GitHub is based on the Git engine, so you still have versioning of your files, but you can now also collaborate with other people (since they can access your folder in GitHub too.

## Signup on GitHub

To get started with GitHub, you will need a GitHub account. Browse to http://www.github.com. You’ll need to enter

- a username (this name must be unique on GitHub and will be visible to all GitHub users),
- an email address (this email address will be used to validate your account and receive notification, your email address will not be visible to all GitHub users)
- a password (the username and the password are used to sign in to GitHub and for contributing)

When you have submitted the values, you can select a plan.

<img src="/images/2016-02-03/01-Plan.png" width="700">

The free plan is the default value and we’ll start with that one. We don’t need an organization yet either, so leave that checkbox unchecked. Click **Finish sign up** on the bottom of the page. You will be redirected to your GitHub start page.

<img src="/images/2016-02-03/02-Start.png" width="700">

Before you start doing anything in GitHub open your email account first. You should have received and email from GitHub for validating your email address. Click the verify email address button in your email. You will be redirected to your start page in GitHub.

## Repositories

Now that we an account we can start. I suggest you start by creating a repository. A repository is a container (a folder) for storing your files. The easiest way to create a repository is to click the + icon on the top right of your start page.

<img src="/images/2016-02-03/03-New-Repo.png" width="700">

Select to create a new repository. You’ll need to specify a name for the repository. That name must be unique within your own account. The description is optional, but it is useful to enter one. This will help other GitHub user to quickly identify the purpose and content of your repository. Be default each repository you create is public. This means that anyone in the world (even without a GitHub account) can see your repository and its content. You can select to make the repository private. A private repository is hidden for anyone but you. You can give access to your private repository to individual GitHub users. But to create a private repository you will need to upgrade your GitHub account to a paid plan. We’ll select public for now. To give a more detailed explanation of your repository you can create a readme file. Later in this blog I’ll look at the readme in more detail. We’ll skip it for now (don’t enable the checkbox).

<img src="/images/2016-02-03/04-New-Repo-name.png" width="700">

When you click **create repository** you are redirected to the repository start page. So… now what? Before we interact with the repository I need to explain in what way we can interact with our web-based repository. You cannot upload a file from the browser into the repository. You will need a client for that.

<img src="/images/2016-02-03/05-New-Repo-default.png" width="700">

## Clients

There are two common clients you can use to interact with GitHub.

- Git is the engine of the local version control system. Git provides a command line interface and is the only tool that allows you to run all Git commands.
- Github Desktop is an application with a graphical UI that allows you to perform all basic Git activities without a command line.

For the purpose of this blog we’ll look at the operations from the perspective of each tool. I’ll describe each procedure for Git console and for Git Desktop. You do not need both clients. Based on your preference you can install and use the client that works best for you.

### Git

To install the Git client browse to <http://www.git-scm.com/> and select to download the latest source release.

<img src="/images/2016-02-03/06-Git-Download.png" width="700">

Start the setup wizard. Accept the License, leave the defaults in the **Select Components** screen.

<img src="/images/2016-02-03/07-Git-Iinstall1.png" width="700">

In the Adjusting your PATH environment select **Use Git from the Windows Command Prompt**

<img src="/images/2016-02-03/08-Git-Iinstall2.png" width="700">

In the **Configuring the line ending conversions* screen select **Checkout as-is, commit as-is**

<img src="/images/2016-02-03/09-Git-Iinstall3.png" width="700">

In the **Configuring the terminal emulator to use with Git Bash** select **Use Windows’ default console window**

<img src="/images/2016-02-03/10-Git-Iinstall4.png" width="700">

Leave the checkmark unchecked on the **Configuring experimental performance tweaks** screen.

<img src="/images/2016-02-03/11-Git-Iinstall5.png" width="700">

After the installation completes open a command prompt. Type Git in the console and hit enter. You’ll notice that the PATH variable has been updated so you are able to run Git directly without changing the folder location (although is still might be useful to do that, as we will see later in this blog).

<img src="/images/2016-02-03/12-Git-commandline.png" width="700">

### Git Desktop

To install GitHub Desktop browse to <https://desktop.github.com/> and select to download GitHub Desktop.

<img src="/images/2016-02-03/13-GitHubD-download.png" width="700">

The installation will start by downloading the rest of the software (just over 100MB). When the file is downloaded and the installation is complete GitHub Desktop will open.

<img src="/images/2016-02-03/14-GitHubD-start.png" width="700">

Before we add a repository we need to configure some additional settings. Open the options by clicking on the icon that is in the right top corner.

<img src="/images/2016-02-03/15-GitHubD-options1.png" width="700">

In the options screen select the **Add account**. In the login screen that opens specify the **GitHub username** and **password** that you created earlier in this blog.

<img src="/images/2016-02-03/16-GitHubD-options2.png" width="700">

Specify an email address in the **Configure git** entry. The Clone path is used to story the local copies of your GitHub content. Make sure the **Clone path** matches the location you want to locally store the templates you will be working on. You will notice later in this blog that you can override this location. You can consider this path as the default value.

<img src="/images/2016-02-03/17-GitHubD-options3.png" width="700">

Save the settings. Now that we have both clients in place we can start working with our repository.

## Clone

The repository we created in our GitHub account is only available on the hosted web service. We need to great a copy of this repository locally first. We do this by cloning the repository. A clone is a moment is time local copy of the repository from GitHub. Changes in the source repository in GitHub or changes in the local repository will not automatically sync.

### Clone a repository with Git

Open the command line on the local machine. When you clone a repository from the command line, Git will create the clone in the current path. First browse to the folder where you want the clone created. Each clone will git its own folder, so you just need a root folder (e.g. c:\templates). Browse to the root folder.

<img src="/images/2016-02-03/18-Clone1.png" width="700">

Next we need lookup the Git endpoint of our GitHub repository. You can find the endpoint by browsing GitHub (if you are not signed in, sign in with the credentials you created earlier). Select our first repository. On the top the endpoint for our repository is displayed. You can click the copy Icon on the right (or directly copy the URI).

<img src="/images/2016-02-03/19-Clone2.png" width="700">

Type `git clone <the repository endpoint>` in you command line. 
```
git clone https://github.com/marcvaneijkdemo/MyFirstRepository.git
```

<img src="/images/2016-02-03/20-Clone3.png" width="700">

We’ll get a warning that we are cloning an empty repository. You can ignore that, because we are doing just that!

If you browse to the current path in Windows Explorer, you will notice that a new folder with the repository name. This folder only contains a hidden .git folder that is used by git for versioning data.

Not too bad right? Just a single command and you created a clone.

```
## git clone <the repository endpoint>
Git clone https://github.com/marcvaneijkdemo/MyFirstRepository.git
```

Now let’s see how cloning in GitHub Desktop works.

### Clone a repository with GitHub Desktop

There are two ways to clone a repository with GitHub Desktop. You can open GitHub Desktop, Click the plus icon on the top left and select clone. The UI connects to your GitHub account (remember you specified your GitHub credentials in GitHub Desktop earlier) and show the repositories on your account. Select the repository you want to clone and click **Clone <RepositoryName>**. You are prompted to select a local path for you clone. The default location is the path we specified in the installation of GitHub Desktop earlier. You can also select a different path. In the folder you select, a subfolder is created with the name of the repository. This folder only contains a hidden .git folder that is used by git for versioning data.

<img src="/images/2016-02-03/21-GitHubD-clone1.png" width="700">

It also possible to browse GitHub, select your repository and click the **Set up in Desktop** button.

<img src="/images/2016-02-03/22-GitHubD-clone2.png" width="700">

The rest of the steps are the same as cloning the repository directly from GitHub Desktop.

Now that we have a local copy of our repository let’s make some changes to it. The best thing to do next is to create a good description for your repository. You do that with a README.md file.

## README.md

If you create a README file in the root of your repository, GitHub will automatically render the content of the file on the start page of your repository. You can create a README in plain text or use markdown. Markdown allows you to enhance your text. Let’s start with a super simple example using plain text. Open Windows Explorer and browse to the folder of your cloned repository. Create a new text file in the folder. Add a description about your repository and save the file. Rename the file, including the file extension, to README.md (README in uppercase and md in lowercase).

<img src="/images/2016-02-03/23-README11.png" width="700">

Now that we made some changes to our local clone, we need to sync that to our repository on GitHub.

## Push

To sync the changes we have made locally to our repository in GitHub we use a command called `Push`. First we will look at the steps to push the changes with the Git client.

### Push with Git

Open a command prompt and change the path to the local repository folder

<img src="/images/2016-02-03/24-GitPush1.png" width="700">

When we cloned the repository Git created a hidden folder called .get in the repository folder. This hidden folder is used by Git for versioning, but also contains some information about the source repository (called origin). We can get the remote endpoint that are currently configured for this local clone.

```
git remote –v
```

<img src="/images/2016-02-03/25-GitPush2.png" width="700">

You will see two entries with the name origin (the source of the clone). A synchronization always is a one way process. One entry is for retrieving updates from GitHub to your local clone (fetch) and one entry is for syncing your local changes to the repository on GitHub (push). Both entries are configured with the public endpoint of your repository. We can use the alias original in our push command. But before we can push the changes to our repository we need to update the local index (in the hidden .git folder) with the changes. You can add changed files individually (be specifying the filename), or add all changes in the folder (by specifying a dot) to the index.

```
git add .
```

<img src="/images/2016-02-03/26-GitPush3.png" width="700">

The changes in the updated index can be recorded in a snapshot of our local clone. The snapshot is called a commit (you are committing the changes). Git provides you with the option to select a commit of the local clone to work with and based on its versioning system, it will revert the local clone to the moment in time snapshot of your folder. You can understand that a commit is imported, because if you made changes to the local clone that are destructive to your work, you can just revert to the last working commit and resume your work from there. When you commit you need to specify a comment. These comments are very important as they will inform you and other GitHub users of the changes you made in this commit when looking at it later, without revisiting all the actual code you changed. Make sure you specify a descriptive comment.

```
git commit -m “added README.md”
```

<img src="/images/2016-02-03/27-GitPush4.png" width="700">

The snapshot we just created is still on the local clone. We can resume working on the local clone, by adding, changing or deleting files and folder. Recording the changes in more commits, by running the same `git commit` command (each with their own comment). All these changes and commits are on your local machine. Nobody but you is able to access the folder and if your machine crashes, all your work is gone. To sync your work to your repository in GitHub we need to `push` the commits. Before we push our changes to GitHub, it is crucial to understand that the content on your GitHub repository might have changed while you were working with the local clone. You might have changes files directly on GitHub or other users might have contributed to your code (which we will cover later). But if you have a file that is changed on GitHub and the same file in your clone that you changed locally, what change is authoritive? Git does not know. To prevent this issue you need to get the latest version from GitHub first before you push your local changes to it. There are two ways to do this with Git. We can either `fetch` and `merge` or we can `pull`. Pull is a combination of (fetch and merge in one action). Pull seems to be the logical thing to do right? But there is a caveat. Let’s explain with an example.

Assume we have a repository with a README.md file already in there. We create a local clone. After we make a change to the README.md file locally.

<img src="/images/2016-02-03/28-GitConflict1.png" width="700">

We also make a change to the file directly on GitHub

<img src="/images/2016-02-03/29-GitConflict2.png" width="700">

Now remember, my GitHub repository and the local clone do not sync automatically. Doing a pull will result a conflict for this file.

<img src="/images/2016-02-03/30-GitConflict3.png" width="700">

You can see that the pull request does an auto-merge of README.md (pull is a combination of fetch and merge). And it notifies us there is a merge conflict in README.md. The merge is performed in the local clone. The content of the README.md is updated by Git with the conflicting values.

<img src="/images/2016-02-03/31-GitConflict4.png" width="700">

The auto-merge will only update the conflicting code within a file. All none conflicting code is not touched.

If you want more control over the merging process you can use the fetch and merge commands individually instead of the combined pull command. I have created a blog with a step by step for git fetch and merge here.

You resolve a conflict by editing the file to manually merge the parts of the file that git had trouble merging. This may mean discarding either your changes or someone else’s or doing a mix of the two. You will also need to delete the `<<<<<<<`, `=======`, and `>>>>>>>` in the file.

<img src="/images/2016-02-03/32-GitConflict5.png" width="700">

Next perform an `git add` and a `git commit`.

<img src="/images/2016-02-03/33-GitConflict6.png" width="700">

We already saw that Git created an alias for the endpoint of our repository called origin. By running the following command we will push the master branch into the origin.

```
git push origin master
```

<img src="/images/2016-02-03/34-GitPush.png" width="700">

You can read this line as push into origin (our source repository on GitHub) the master. The master you say? What is the master? The master is the name of a branch. We’ll cover branches later in this blog. Just know for now that each repository has a branch called master and could potentially contain more branches.

If we now take a look at our repository on GitHub we can see that the start page has been updated. It shows the content of the repository (the README.md file) and it also displays the content of the README.md file directly below the contents of the repository (with the changes we made locally). The endpoint of the repository (we used for creating the clone with Git) and the download to GitHub desktop button are moved to the top of the repository). The comment is displayed for the files that where changed in this commit.

<img src="/images/2016-02-03/35-GitConflict7.png" width="700">

It might seem like a lot of steps to go through. But if we summarize the steps, it’s just four lines of code.

```
Git add .
Git commit -m “your comment about the changes”
Git pull origin master
Git push origin master
```

### Push with Github Desktop

GitHub Desktop provides you with a UI to perform the same steps for committing and pushing your changes. After you create the README.md file in your empty local clone, the GitHub Desktop will pick up these changes and display them in the client.

<img src="/images/2016-02-03/36-GitHubD-Push1.png" width="700">

You can even click on the file and the changes you made will be visible (in green what you added and in red what you deleted)

<img src="/images/2016-02-03/37-GitHubD-Push2.png" width="700">

There is no need to add the files to the index. GitHub client will do that for you. You will need to commit and push the changes. All you need to do to make a commit, after you made changes to the local clone, is to specify a title (comment) and a description for the commit and click “Commit to master”.

<img src="/images/2016-02-03/38-GitHubD-Push3.png" width="700">

On the top right of the GitHub desktop you will see a new moment in time (commit) in the timeline. Clicking on a circle in that “timeline” shows you the changes made in that commit. Clicking on the last icon (most right circle), presents the current state.

<img src="/images/2016-02-03/39-GitHubD-Sync.png" width="700">

The last step to take is to push the commit to GitHub. You can do that by clicking the **Sync** button on the top right of GitHub Desktop. If it is the first time you are pushing to a new repository, the button will be called **Publish**. For any subsequent pushes the button is called Sync.

<img src="/images/2016-02-03/40-GitHubD-Publish.png" width="700">

There is an important difference with the command line Git client here. Clicking the Sync button will not only push the local changes to GitHub, but it will first pull possible changes from GitHub first. These possible changes might be caused by you editing files in the repository directly in GitHub. If that is the case, GitHub will notify you that a conflict exists.

<img src="/images/2016-02-03/41-GitHubD-Conflict1.png" width="700">

It will then show what files has a conflict and what the conflict is.

<img src="/images/2016-02-03/42-GitHubD-Conflict2.png" width="700">

If we look at the content of the file, we can see that the conflicts has been marked in the content of the file too.

<img src="/images/2016-02-03/43-GitHubD-Conflict3.png" width="700">

We can solve this conflict, the same way as described in the Git procedure. Edit the file to manually to merge the parts of the file that GitHub had trouble merging. This may mean discarding either your changes or someone else’s or doing a mix of the two. You will also need to delete the `<<<<<<<`, `=======`, and `>>>>>>>` in the file.

<img src="/images/2016-02-03/44-GitHubD-Conflict4.png" width="700">

Save the file. GitHub will pick up the new changes.

<img src="/images/2016-02-03/45-GitHubD-Conflict5.png" width="700">

Make a new commit.

<img src="/images/2016-02-03/46-GitHubD-Conflict6.png" width="700">

And sync to GitHub by clicking the **Sync** button on the top right. Note the icon of the commit is different, because of the merge.

<img src="/images/2016-02-03/47-GitHubD-Conflict7.png" width="700">

The Sync now completes successfully and the README.md file on the GitHub repository contains the final edits we just made locally.

<img src="/images/2016-02-03/48-GitHubD-Conflict8.png" width="700">

There is much more possible with the commit, push and pull commands, but this should give you enough for basic operations. It is very important to commit and push frequently. That will ensure your work is safe.

<img src="/images/2016-02-03/49-fire.jpg" width="700">

## Enhance the README

By adding a README.md file, our GitHub repository is updated with a simple description. Besides adding a more detailed description, it is also a good idea to explain in what way other people on GitHub should contribute to your repository. Do you have any naming conventions, folder structure requirements, etc. These details help other contributors to prepare their local work in line with your guidelines. Besides adding these details to you README.md you can also use markdown to enhance the display of your content. You can find a quick guide to markdown here <https://guides.github.com/features/mastering-markdown/>

After you updated the README.md file locally, you can repeat the same procedure (with either Git or GitHub Desktop) to update the file to your GitHub repository.

## Fork

It is possible to create a copy of the repository from another GitHub account in your own GitHub account. This copy is called a fork. A fork is a moment is time copy created on GitHub. When you fork a repository the reference to the original repository is recorded in the fork and in the source repository. Since the fork is in your own account, you have the permissions to push to it directly. This allows you to make changes to the code without interfering with the source repository.

To fork a repository browse to the repository in another GitHub account. Click the **Fork** icon on the right top of the repository start page.

<img src="/images/2016-02-03/50-Fork1.png" width="700">

Creating the fork in your account takes a couple of seconds. After the fork is created you are redirected to the fork in your account. You can see that the reference to the original repository is recorded as part of the fork.

<img src="/images/2016-02-03/51-Fork2.png" width="700">

A fork is a moment in time copy created in your GitHub account, before you can edit the files you need to clone the fork to your local machine. The steps to clone the fork to your local machine are exactly the same as the steps described earlier to clone a repository.

## Pull request

In the examples used so far you are the author of all the content and it is stored in your GitHub account that you have full access to. What makes GitHub really interesting is that you are able to contribute to repositories of other GitHub users and other GitHub users are able to contribute to your repositories. The primary difference between your repository and a repository of another GitHub account, is that you have access to push commits into your repository and you do not have that access to repository in another GitHub account.

Instead of pushing commits into a repository of another GitHub account, you submit a pull request. The owner of the GitHub account gets notified by GitHub that there is a pull request, he is able to evaluate the changes in the commits of the pull request. He can then choose to merge the pull request, request you to make additional edits to your code or decline the pull request.

To start contributing to a repository in another GitHub account you need to fork that repository to your account, clone the fork to your local machine, add files/folders or updates to the existing files, commit and send a pull request to merge your updates to the source directory.

### Pull request with Git

Start by creating a fork of the repository you want to contribute to. Then create a clone of the fork on your local machine. Get the repository endpoint by browsing to the start page of the fork in your GitHub account.

<img src="/images/2016-02-03/52-PR1.png" width="700">

Copy the endpoint URI and open a command prompt. Change the current path to the directory where you want the clone directory to be created and run

```
Git clone <endpoint of the fork>
```

This will make a local clone of the fork. Just as with the clone of your own repository Git creates to aliases (one for `fetch` and one for `push`) called **origin** pointing to the endpoint of the repository on GitHub.

<img src="/images/2016-02-03/53-PR2.png" width="700">

But we do not have any reference to the source (also called upstream) repository. To add a reference to the upstream repository run

```
## git remote add upstream <endpoint of the upstream repository>
git remote add upstream https://github.com/marcvaneijk/RepositoryInAnotherAccount.git
```

<img src="/images/2016-02-03/54-PR3.png" width="700">

When you now do a `Git remote -v` you will notice you two additional aliases (`fetch` and `push`) called **upstream**. You can use `fetch` (or `pull` as explained earlier) to update your clone. You cannot use `push` to push changes directly to the upstream server. You will need to submit a pull-request for that.

Let’s make a change locally. I’ll add a file to the local clone of the fork.

<img src="/images/2016-02-03/55-PR4.png" width="700">

The procedure to contribute to the repository of the other GitHub user is almost the same as contributing to you own repository.

```
Git add .
Git commit -m “Added initial version of NewFile.txt”
```

<img src="/images/2016-02-03/56-PR5.png" width="700">

Instead of pulling the latest updates from our fork (which we have not changed), we will get latest updates of the upstream repository.

```
Git pull upstream master
```

<img src="/images/2016-02-03/57-PR6.png" width="700">

Now in this case it is much more likely that code in the upstream is changed. Because other people are contributing to it as well.

We solve the conflict in the exact same way as was explained earlier in this blog. Next we need to update our fork with the changes with the push command (this push contains our edits and the last update from the upstream).

```
Git push origin master
```

<img src="/images/2016-02-03/58-PR7.png" width="700">

To summarize the code we just executed

```
# Fork the repository from another GitHub account
Git clone <endpoint of the fork>
Git remote add upstream <endpoint of the upstream repository>

# Make changes
Git add .
Git commit -m “Added initial version of NewFile.txt”
Git pull upstream master

# Solve potential conflicts, rerun Git add and Git commit
Git push origin master
```

The last step we need to perform is to create a pull request. When working with Git command line we will make that pull request directly from GitHub. Open a browser, browse to <github.com> and sign in with your github credentials. Browse to the fork we just pushed the changes to.

<img src="/images/2016-02-03/59-PR8.png" width="700">

On the start page of the fork, click the green **New pull request** icon. That will open a page to compare the differences between the fork and the upstream repository.

<img src="/images/2016-02-03/60-PR9.png" width="700">

You can see that the is one new file (called NewFile.txt) that is on the fork, but not on the upstream. Click the green **create pull request** button.

<img src="/images/2016-02-03/61-PR10.png" width="700">

Specify a title and explanation that will help the author of the upstream repository to make a good assessment of the code he might be merging into his code. Click create pull request.

You are now redirected to the account of the upstream repository. The page opens in your pull request.

<img src="/images/2016-02-03/62-PR11.png" width="700">

The owner of the upstream repository is notified that a new pull-request is awaiting his review. This process (`fork`, `clone`, `commit`, `push` and `pull-request` is complete integrated within GitHub Desktop.

### Pull request with GitHub Desktop

Start by creating a fork of the repository you want to contribute to. Then create a clone of the fork on your local machine You can either click on the Download to GitHub Desktop button on start page of the fork in your GitHub account

<img src="/images/2016-02-03/63-GH-PR1.png" width="700">

Or you create the local clone of the fork directly from GitHub Desktop.

<img src="/images/2016-02-03/64-GH-PR2.png" width="700">

After the repository is opened there is no need to add the upstream server. GitHub Desktop will do that automatically.

Let’s make a change locally. I’ll add a file to the local clone of the fork.

<img src="/images/2016-02-03/65-GH-PR3.png" width="700">

The change is picked up by GitHub Desktop

<img src="/images/2016-02-03/66-GH-PR4.png" width="700">

Commit the change to the master.

<img src="/images/2016-02-03/67-GH-PR5.png" width="700">

And sync (`pull` and `push`) the changes to the fork in GitHub.

<img src="/images/2016-02-03/68-GH-Sync.png" width="700">

Note that the commits to the fork are displayed in a separate timeline to the upstream repository. Instead of doing a pull request from GitHub, we can also perform a pull request directly from GitHub Desktop. On the top right an icon called Pull request is displayed.

<img src="/images/2016-02-03/69-GH-PR7.png" width="700">

If we click that icon a windows pops up that allows you to submit a pull request to the upstream repository.

<img src="/images/2016-02-03/70-GH-PR8.png" width="700">

When the pull request completes, GitHub Desktop provides you with a link to see the pull request on GitHub

<img src="/images/2016-02-03/71-GH-PR9.png" width="700">

This will bring up the same page as we saw earlier, when we submitted the pull request directly on GitHub.

## Branches

You have seen me mention the master numerous times throughout this blog. You can think of a branch as an independent line of development. There is always one default branch in a repository. You can create multiple branches. This if useful when you are working a specific feature of the total project. You create a branch of the master for the changes to the feature. You can push and test your commits to the branch, without touching the original code in the master branch. When your code is complete and tested, you can merge the branch into the master.

### Branching with Git

Open a command prompt change the current path to your local clone. In this example we will use the MyFirstRepository clone. The see all the available brances in this clone run

```
Git branch
```

<img src="/images/2016-02-03/72-GitBranch1.png" width="700">

Since we have not create any additional braches yet, the master is the only branch. It is also the active branch, hence it is displayed in green. To create a new branch run the following command

```
# Git branch <name of the branch>
Git branch myfeature
```

<img src="/images/2016-02-03/73-GitBranch2.png" width="700">

This creates a new branch called my feature. If we run the command to retrieve all the branches, we can see that the new branch called myfeature now also exists, but master is still the current branch. To select another branch run the following command

```
## Git checkout <name of the branch you want to make current>
Git checkout myfeature
```

<img src="/images/2016-02-03/74-GitBranch3.png" width="700">

Please note that you are note able to checkout to another branch without committing your changes to the current active branch first. If we now retrieve the braches again

```
Git branch
```

<img src="/images/2016-02-03/75-GitBranch4.png" width="700">

We can see that the myfeature branch is the current active branch. To give you an idea of the independent development line each branch provides, we’ll create a new file in the current active branch. In this example I have created a .txt file called NewTextFileInBranch.

<img src="/images/2016-02-03/76-GitBranch5.png" width="700">

Next we need to add and commit the changes in our branch.

```
Git add .
Git commit -m “new textfile in branch”
```

<img src="/images/2016-02-03/77-GitBranch6.png" width="700">

Now that we have comitted the changes to our branch, lets change the branch to master.

```
Git checkout master
```

<img src="/images/2016-02-03/78-GitBranch7.png" width="700">

After the current branch has been configured to master, the folder is in its original state (we did not make any changes to the master).

<img src="/images/2016-02-03/79-GitBranch8.png" width="700">

If we were to change the branch to myfeature again. The NewTextFileInBranch.txt would be present again. Each branch will have its own version of the truth.

### Branching with GitHub Desktop

GitHub Desktop also allows you to create branches. Open GitHub Desktop and select a repository. For this example we will use the **RepositoryInAnotherAccount**. Which is the local clone of the fork we used for the example with the pull-request.On the top menu the is an account (next to the name of the branch). If you hover over that icon it will say **Create a new branch**. Click that icon.

<img src="/images/2016-02-03/80-GitHBranch1.png" width="700">

Specify the name of the branch. Use something descriptive (for example the name of the folder containing your ARM template).

<img src="/images/2016-02-03/81-GitHBranch2.png" width="700">

When you create the new branch GitHub Desktop will automatically checkout to that new branch. Make some changes to the local clone, commit these changes. Make additional changes to the local clone and make do a commit again. You now multiple commits that are local only. Sync the commits to your fork and open a pull request.

<img src="/images/2016-02-03/83-GitHBranch3.png" width="700">

When you browse to the pull request on GitHub, you can now see that the commits are grouped together in a single pull request, but still consists of individual commits. This makes is more clear for the author to evaluate the changes and merge the content.

<img src="/images/2016-02-03/84-GitHBranch4.png" width="700">

## Summary

The public GitHub repositories with ARM templates usually have a folder per deployment to improve the repository structure. If you are contributing to a public repository the following workflow is strongly advised.

- Create a fork
- Clone the fork locally
- Create a branch for a deployment you a recontributing to
- Select the branch
- Make your changes locally, commit on edit type
- Push to your fork
- Do a pull request
- If you want to contribute on another deployment, create a new branch and select it.

It was a long post, but I hope that it will save you a lot of time trying to fit the pieces togheter. Now it’s time to make some contributions on GitHub. Have fun!
