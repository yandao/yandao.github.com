---
layout: post
title: "Create a Git repository from an empty SVN repository"
description: ""
category: quick tips
tags: [git, svn]
---
{% include JB/setup %}


Something that is unlikely to happen, unless probably when you need to test some setups. Still, might be useful to keep.

## The steps

    # Create SVN repo
    $ svnadmin create svn_repo
    $ svn mkdir file:///full/path/to/svn_repo/{trunk,tags,branches} -m "Initial directory structure"

    # Initialise a Git repo from the empty SVN repo
    $ git svn init --trunk=trunk <URL_to_svn_repo>
    $ git svn fetch
    $ git rebase --onto remotes/origin/trunk --root master


## Reference

* [How to commit a Git repo to an empty repo SVN server?](http://stackoverflow.com/a/981765)
