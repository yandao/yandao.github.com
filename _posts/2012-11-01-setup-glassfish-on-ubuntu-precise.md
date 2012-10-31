---
layout: post
title: "Setup GlassFish on Ubuntu 12.04 (Precise Pangolin)"
description: ""
category: configuration 
tags: [ubuntu, java, glassfish]
---
{% include JB/setup %}

## Introduction ##

Setting up a fresh GlassFish server can be a nightmare. This post attempts to consolidate all the 


## Install Java JDK ##

A fresh installation of an Ubuntu server do not include Java JDK by default due to some licensing issue with Oracle (I will refrain from making any comments here). The official method of installing Java JDK is by running the binaries provided from the [Oracle Java website](http://www.oracle.com/technetwork/java/javase/downloads/index.html). Thankfully, there are always some [3rd party efforts](http://www.webupd8.org/p/ubuntu-ppas-by-webupd8.html) that make it possible to install Java JDK via apt-get. With all the reports on security vulnerabilities on Java 6, the recommended solution nowadays is to install Java 7 directly.

### The Steps ###

	$ sudo apt-get install build-essential python-software-properties unzip

	$ sudo add-apt-repository ppa:webupd8team/java
	$ sudo apt-get update
	$ sudo apt-get install oracle-java7-installer

It may be good to install OpenJDK too for testing the more "bleeding-edge" features of Java:

	$ sudo apt-get install openjdk-7-jdk

To list all the Java JDKs/JREs that are installed:

	$ sudo update-java-alternatives -l

To set Oracle Java as default:

	$ sudo update-java-alternatives -s java-7-oracle

Since we install the JDK via apt-get, we can remove it via apt-get too. If you wish to remove the PPA too:

	$ sudo ppa-purge ppa:webupd8team/java


## Install GlassFish ##

GlassFish can be installed using an installer or as a standalone zip file, both of which are available from the [Downloads](http://glassfish.java.net/public/downloadsindex.html) page. For simplicity, we choose the zip file. The latest version as of this writing is 3.1.2.2. Typically, I place all the applications that can be run without root privileges in ~/opt. **From the point of security, it may be good to create a new user and group specifically for running GlassFish processes only**. The steps can be found [here](/configuration/2012/10/11/first-things-after-fresh-installation-of-a-linux-server/#add-non-root-user) and therefore will not be included in this post.

### The Steps ###

	$ mkdir -p ~/opt
	$ cd ~/opt/
	$ wget http://download.java.net/glassfish/3.1.2.2/release/glassfish-3.1.2.2.zip
	$ unzip glassfish-3.1.2.2.zip 

It is a good idea to include the bin/ path of GlassFish to the PATH environment variable, so that we can run asadmin from anywhere:

	$ vim ~/.bash_profile

		export PATH=$PATH:~/opt/glassfish3/bin


## Initial Configuration for GlassFish ###

Just a few things to do first after the fresh installation of GlassFish. This will save us from all the hair-pulling and head-banging in the future.

### Disable auto-update ###

The reason why GlassFish is always so slow when starting is because it tries to perform an automatic update first before starting the server proper. To disable this, include the following JVM option in the domain.xml file (just place it together with all the other options):

	$ vim ~/opt/glassfish3/glassfish/domains/domain1/config/domain.xml

		...

		<jvm-options>-Dcom.sun.enterprise.tools.admingui.NO_NETWORK=true</jvm-options>

		...

Now GlassFish will start up in blazing fast time, and the Admin Console will be much responsive too.

### Change Admin password ###

Starting from version 3.1.2, any administration processes will require entering of admin username and password. I could not, for the life of me, find the default admin password for GlassFish (it is probably empty string). Anyway, this [article](https://blogs.oracle.com/dipol/entry/glassfish_3_1_2_secure) outlines the steps to change the admin password using asadmin:

	$ cd ~/opt/glassfish3/glassfish/
	$ touch /tmp/password.txt
	$ chmod 600 /tmp/password.txt 
	$ echo "AS_ADMIN_PASSWORD=" > /tmp/password.txt 
	$ echo "AS_ADMIN_NEWPASSWORD=<the_new_password>" >> /tmp/password.txt 
	$ asadmin --user admin --passwordfile /tmp/password.txt change-admin-password --domain_name domain1
	$ rm /tmp/password.txt 

Now GlassFish is set up with a new admin password.

To enable remote administration of GlassFish server:

	$ asadmin enable-secure-admin

At this point, we can start the GlassFish server already:

	$ asadmin start-domain

and watch how blazing fast it starts up without the auto-update running in the background!
