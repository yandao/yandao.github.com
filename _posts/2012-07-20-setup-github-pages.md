---
layout: post
title: "Setup GitHub pages"
description: ""
category: configuration
tags: [github, blog]
---
{% include JB/setup %}


# Motivation #

I have been looking for a tool that will allow me to write documentation/technical posts (preferably using Markdown), and easily generate static HTML pages from the source documents. The site must also be able to generate an index of all documents automatically. Even though Jekyll is intended to be used for writing blogs, it is generally good enough to be used for writing documentation, and fulfills my basic requirements for a documentation site.

Jekyll comes with a built-in web server that allows you to run your documentation site locally. Alternatively, you can have it hosted for free on GitHub Pages too, with the additional benefit of using git for version control of your contents. A simple "git push" and the new contents will become live shortly.

This post attempts to document the steps for setting up a Jekyll site that is hosted on GitHub Pages.


# Create GitHub Pages Repository #

The first step is to create a repository on GitHub for your GitHub Pages. It is basically the same as creating other repositories, except that the repository name must be called **&lt;username&gt;.github.com**.


# Setup a Jekyll Site #

Instead of building from scratch, it is generally a good idea to just make use of many great layouts out there, and customise it accordingly. I use [Jekyll Bootstrap]((http://jekyllbootstrap.com) layout for this site, and I am pretty much ready to go except for a few minor changes.

The steps to create a Jekyll site using Jekyll Bootstrap and have it hosted on GitHub Pages are listed below:

		$ git clone https://github.com/plusjade/jekyll-bootstrap.git <username>.github.com
		$ cd <username>.github.com
		$ git config user.name "Your Name"
		$ git config user.email "your_email@youremail.com"
		$ git config helper.credential osxkeychain
		$ git remote set-url origin https://github.com/<username>/<username>.github.com.git
		$ git push origin master

Wait for about 10 minutes or so, and the site will be live on http://&lt;username&gt;.github.com. Subsequent additions and changes are as simple as committing and pushing them to the GitHub repository.

*NOTE*:  
The minor changes to Jekyll Bootstrap used in this site are documented [here](/configuration/2012/07/20/jekyll-bootstrap-setup). They are mainly cosmetic changes, with one additional jQuery plugin for generating Table of Contents.

# References #

* [GitHub Pages Help](https://help.github.com/categories/20/articles)
* [Jekyll Bootstrap](http://jekyllbootstrap.com)