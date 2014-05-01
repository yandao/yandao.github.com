---
layout: post
title: "Provisioning JDK on Vagrant with Puppet"
description: ""
category: configuration
tags: [vagrant, virtualbox, java]
---
{% include JB/setup %}


## Motivation

Obviously, we can always create our own base box with JDK already installed, run **"vagrant init &lt;base-box-with-jdk&gt;; vagrant up"**, and we are good to go. However, on a basic Ubuntu server setup, the size of the disk image doubles (~375 MB to ~800 MB) with JDK included. So, depending on the need, it might be a good idea to use Puppet to run the required steps for installing JDK on a freshly-installed Ubuntu server.

As a refresher, the steps are as follows:

    $ sudo apt-get install build-essential python-software-properties
    $ sudo add-apt-repository ppa:webupd8team/java
    $ sudo apt-get update
    $ sudo apt-get install oracle-java7-installer

**NOTE**: To install JDK 8, change the package name to **oracle-java8-installer**.

## Translating into Puppet

Basically, we will need to translate the four lines above into a Puppet manifest, and then enable provisioning with Puppet on Vagrantfile.

A simple translation is shown below, in a manifest file called **default.pp**:

    # default.pp - basically translating the four lines below in Puppet.

    # $ sudo apt-get install build-essential python-software-properties
    # $ sudo add-apt-repository ppa:webupd8team/java
    # $ sudo apt-get update
    # $ sudo apt-get install oracle-java7-installer

    package {
        'build-essential':
            ensure => installed;
        'python-software-properties':
            ensure => installed;
    } ->
    exec { 'add-webupd8-repo':
        command => 'sudo add-apt-repository ppa:webupd8team/java',
        path => [ '/bin/', '/sbin/', '/usr/bin/', '/usr/sbin/' ];
    } ->
    exec { 'apt-update':
        command => 'sudo apt-get update',
        path => [ '/bin/', '/sbin/', '/usr/bin/', '/usr/sbin/' ];
    } ->
    exec {
        'set-licence-selected':
            command => 'echo debconf shared/accepted-oracle-license-v1-1 select true | sudo debconf-set-selections',
            path => [ '/bin/', '/sbin/', '/usr/bin/', '/usr/sbin/' ];
        'set-licence-seen':
            command => 'echo debconf shared/accepted-oracle-license-v1-1 seen true | sudo debconf-set-selections',
            path => [ '/bin/', '/sbin/', '/usr/bin/', '/usr/sbin/' ];
    } ->
    package { 'oracle-java7-installer':
        ensure => present;
    }

Enable provisioning with Puppet on Vagrantfile (refer to earlier post for the full Vagrantfile content):

    ...

    # Enable provisioning with Puppet stand alone. Puppet manifests
    # are contained in a directory path relative to this Vagrantfile.
    # You will need to create the manifests directory and a manifest in
    # the file default.pp in the manifests_path directory.
    #
    config.vm.provision "puppet" do |puppet|
      puppet.options = "--verbose --debug"
      puppet.manifests_path = "manifests"
      puppet.manifest_file  = "default.pp"
    end

    ...

Based on the configuration above, save **default.pp** in a sub-directory called **manifests/**.

## Done
