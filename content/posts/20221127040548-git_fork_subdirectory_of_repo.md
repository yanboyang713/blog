---
title: "Git Fork Subdirectory Of Repo"
tags: ["git", "fork"]
draft: false
---

Fork A Subdirectory Of Repo


## Introduction {#introduction}

Ever wanted to fork a subdirectory and not the whole [Git]({{< relref "20230220223747-git.md" >}})/GitHub repository. Well I have, I recently had to fork a subdirectory of one of the repositories I wanted to work on without the need to fork the whole repository. In this post, I will show you how it’s done.


## Clone the repo {#clone-the-repo}

```console
git clone https://github.com/<someones-username>/<some-repo-you-want-to-fork>
cd some-repo-you-want-to-fork
```


## Create a branch using the git subtree command for the folder only {#create-a-branch-using-the-git-subtree-command-for-the-folder-only}

```console
git subtree split --prefix=src -b dir-you-want-to-fork
git checkout dir-you-want-to-fork
```


## Reference List {#reference-list}

1.  <https://blog.mphomphego.co.za/blog/2021/02/07/How-to-fork-a-subdirectory-of-repo-as-a-different-repo-on-GitHub.html>
2.  <https://www.mendelowski.com/docs/git/fork-subdirectory/>
