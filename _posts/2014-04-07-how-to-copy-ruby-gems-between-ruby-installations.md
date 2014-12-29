---
layout: post
title: "How to Copy Ruby Gems between Ruby Installations"
description: ""
category: quick tips
tags: [ruby]
---
{% include JB/setup %}

## Introduction

Yes, there is the copy function for RVM gemsets, but in case it fails for whatever reasons (it didn't work for me), we can try to export the list of gems (minus the version numbers) from the old gemset to a text file, and then read it line by line and pass it as the argument for the gem installation.

    $ rvm use <old_gemset>
    $ gem list | tail -n+4 | awk '{print $1}' > gemlist
    $ rvm use <new_gemset>
    $ cat gemlist | xargs sudo gem install
    $ rm gemlist

## Reference

* [Copy Ruby Gems Between Ruby Installations/Computers/Systems](http://www.williambharding.com/blog/rails/copy-ruby-gems-between-ruby-installationscomputerssystems/)