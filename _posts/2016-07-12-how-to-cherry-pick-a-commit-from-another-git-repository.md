---
layout: post
title: "How to Cherry pick a Commit from Another Git Repository"
description: ""
category: tutorial
tags: [git]
---
{% include JB/setup %}


## Background

I had a small project that was initially included as part of a bigger project. Since it ran separately on its own, and did not use any dependencies from the parent project, I decided to move it into its own Git repository.

However, I also did not want to lose the revision history of that project, so it was a good thing to know that Git has the ability to cherry-pick commit(s) from a repository to another. The following section describes the steps I took to move a subproject to its own Git repository, while maintaining its revision history from the old repository.


## The Steps

Create a new directory for the project

    $ mkdir -p <project_name>
    $ cd <project_name>

**(For reference only)** Tried [these steps](http://stackoverflow.com/a/11477795) initially, didn't work for me though:

    $ git remote add <old_repo_name> <path_to_old_repo>
    $ git fetch <old_repo_name>
    $ git cherry-pick <sha_of_commit_from_old_repo>

Fortunately, [this one-liner](http://stackoverflow.com/a/9507417) works:

    $ git --git-dir=<path_to_old_repo>/.git format-patch -k -1 --stdout <sha_of_commit> | git am -3 -k

The idea is basically to create a patch from the other repository, and applying it to the local repository.


## Footnote

Obviously, if you have many commits to cherry-pick, then it is not very practical, but for a small project with only a few commits to copy over, it is definitely very useful for retaining its revision history.