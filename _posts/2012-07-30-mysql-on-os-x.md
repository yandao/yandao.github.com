---
layout: post
title: "MySQL on OS X"
description: ""
category: configuration
tags: [mysql]
---
{% include JB/setup %}

# Introduction #

Getting up and running with MySQL on OS X for development purposes.

# Installation #

Use Homebrew and save all the tears.

	$ brew install mysql

# Post-Installation Configuration #

	$ export PATH=$PATH:/usr/local/Cellar/mysql/<version>/bin
	$ mysql_install_db --verbose --user=`whoami` --basedir="$(brew --prefix mysql)" --datadir=/usr/local/var/mysql --tmpdir=/tmp

# Start MySQL Server #

	$ /usr/local/Cellar/mysql/<version>/support-files/mysql.server start

# Setup Root User #

	$ mysqladmin -u root password 'password'

# Create Database #

	$ mysql -u root -p

	mysql> create database <database>;
	mysql> grant all on <database>.* to '<some_user>'@'localhost' identified by '<password>';
	mysql> flush privileges;
	
# Use Database #

	mysql> show databases;
	mysql> use <database>;
	mysql> show tables;
	mysql> SELECT * FROM <table>;

# Reference #

[MySQL on Lion with Homebrew](http://www.andrewsavory.com/blog/2011/2144)