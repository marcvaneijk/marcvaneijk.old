---
layout: post
date: 2016-02-17
author: Marc van Eijk
title: Azure Resource Manager – Git Fetch, Diff and Merge
tags:
---
As a follow up to my earlier blog [Azure Resource Manager – Getting started with GitHub](/2016/02/03/githubstart), I got asked how to look at the differences between a repository in GitHub and your local clone before merging all the changes.

As an alternative to Git pull you can also perform the individual commands (fetch and merge). This will allow you to get the changes, evaluate potential conflicts and solve them first, before you merge. Let’s start with the same example as in Azure Resource Manager – Getting started with GitHub. We have a repository in GitHub with a README.md file already in there and we create a local clone. After we make a change to the README.md file locally, we also make a change to the file directly on GitHub.

<img src="/images/2016-02-17/01-LocalEdit.png" width="600">

Instead of pull (which does the fetch and merge), we can just fetch the changes from the origin.

```
Git fetch origin master
```

<img src="/images/2016-02-17/02-Fetch.png" width="600">

The fetch command does not make any changes to your local clone yet. You can first evaluate the changes by running

```
Git diff master origin
```

<img src="/images/2016-02-17/03-Diff.png" width="600">

This will compare your master branch to the origin (your repository in GitHub). As you can see from the output we can now first solve the conflict manually, do a Git add and a commit and then merge. If you try to merge without solving the conflict first by running

```
Git merge origin master
```

<img src="/images/2016-02-17/04-MergeFailed.png" width="600">

You will end up in the same position as the pull request. When you open the README.md file you’ll notice the same issue.

<img src="/images/2016-02-17/05-Merge1.png" width="600">

You resolve a conflict by editing the file to manually merge the parts of the file that Git had trouble merging. This may mean discarding either your changes or someone else’s or doing a mix of the two. You will also need to delete the ‘<<<<<<<‘, ‘=======’, and ‘>>>>>>>’ in the file.

<img src="/images/2016-02-17/06-Merge2.png" width="600">

Next perform an `Git add` and a `Git commit`.

<img src="/images/2016-02-17/07-Commit.png" width="600">

After pushing your clone to GitHub you have successfully merged the local changes and the repository is in sync with your local clone. By using `Git Fetch`, `Diff` and `Merge` you are able to interact with your files at each step. `Git Pull` performs these steps in a single action.
