---
layout: post
title: "Make Git and SVN play nice together"
description: ""
category: tutorial 
tags: [git, svn]
---
{% include JB/setup %}


## Pre-requisites

Install git and svn

    $ brew install git svn

Nice to have GUI tools for accessing git and svn repos

    $ brew cask install laullon-gitx svnx


## Initial step

It is probably a good idea to create a writable clone of the SVN repo that you can mess around with, until you are comfortable with pulling commits from the SVN server and pushing commits from your local git repo.

    $ mkdir -p ~/tmp/test-svn
    $ svnadmin create ~/tmp/test-svn
    $ vim ~/tmp/test-svn/hooks/pre-revprop-change

        #!/usr/bin/env bash

        exit 0;

    $ chmod +x ~/tmp/test-svn/hooks/pre-revprop-change

Initialise, sync and serve.

    $ svnsync init file:///<root_path_to_svn_repo>/test-svn <remote_svn_repo_url>
    $ svnsync sync file:///<root_path_to_svn_repo>/test-svn

    $ svnserve -d --root <root_path_to_svn_repo>


## Create the git clone

    $ git svn clone svn://localhost/test-svn -s

or clone only a subset of the repository (shallow cloning):

    $ git svn clone -r<svn_rev_num>:HEAD svn://localhost/test-svn -s

Now we have a git repo that is fully interoperable with the main svn repo.


## Commit changes to SVN from Git repo

Do a rebase first to resync with the SVN server.

    $ git svn rebase

Now we can push the changes to the SVN server.

    $ git svn dcommit --username <username>

NOTE: --username switch may be optional in some cases.


## References

* [Git and Subversion](http://git-scm.com/book/en/v1/Git-and-Other-Systems-Git-and-Subversion)
* [Git SVN workflow](http://andy.delcambre.com/2008/03/04/git-svn-workflow.html)
