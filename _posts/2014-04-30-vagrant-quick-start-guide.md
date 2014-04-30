---
layout: post
title: "Vagrant Quick Start Guide"
description: ""
category: configuration
tags: [vagrant, virtualbox]
---
{% include JB/setup %}


**NOTE**: This guide basically lists down the steps to get Vagrant up and running, without the explanations. The purpose is to quickly help recall the steps without having to wade through the details.


## The Steps

### Install VirtualBox and Vagrant

* [Download VirtualBox](https://www.virtualbox.org/wiki/Downloads)
* [Download Vagrant](http://downloads.vagrantup.com)

Click next, next and next...

### Setup Vagrant Box

Various sites that provide vagrant boxes:

* [Vagrant Cloud](https://vagrantcloud.com)
* [Vagrantbox.es](http://www.vagrantbox.es)
* [Ubuntu Cloud Images](https://cloud-images.ubuntu.com/vagrant)

Create a directory that will contain all the images:

    $ mkdir -p vagrant
    $ cd vagrant/

Add a Vagrant base box, e.g. Ubuntu Trusty Tahr server daily build:

    $ vagrant box add <box_name> https://cloud-images.ubuntu.com/vagrant/trusty/current/trusty-server-cloudimg-amd64-vagrant-disk1.box

**Tip**: Give a meaningful name for the Vagrant box to quickly identify its content setup, e.g. trusty64-daily etc.

Init the box:

    $ mkdir -p <vm_name>
    $ cd <vm_name>
    $ vagrant init <box_name>

Now we are ready to run the VM on VirtualBox:

    $ vagrant up


### Modify the Vagrantfile

Uncomment and modify the relevant portion in Vagrantfile:

    # -*- mode: ruby -*-
    # vi: set ft=ruby :

    # Vagrantfile API/syntax version. Don't touch unless you know what you're doing!
    VAGRANTFILE_API_VERSION = "2"

    Vagrant.configure(VAGRANTFILE_API_VERSION) do |config|
      # All Vagrant configuration is done here. The most common configuration
      # options are documented and commented below. For a complete reference,
      # please see the online documentation at vagrantup.com.

      # Every Vagrant virtual environment requires a box to build off of.
      config.vm.box = "base-box-name"

      # Create a private network, which allows host-only access to the machine
      # using a specific IP.
      config.vm.network "private_network", type: "dhcp"

      # Share an additional folder to the guest VM. The first argument is
      # the path on the host to the actual folder. The second argument is
      # the path on the guest to mount the folder. And the optional third
      # argument is a set of non-required options.
      config.vm.synced_folder '.', '/vagrant', nfs: true

      # Provider-specific configuration so you can fine-tune various
      # backing providers for Vagrant. These expose provider-specific options.
      config.vm.provider "virtualbox" do |vb|
        host = RbConfig::CONFIG['host_os']

        # Give VM 1/4 system memory & access to all cpu cores on the host
        if host =~ /darwin/
          cpus = `sysctl -n hw.ncpu`.to_i
          # sysctl returns Bytes and we need to convert to MB
          mem = `sysctl -n hw.memsize`.to_i / 1024 / 1024 / 4
        elsif host =~ /linux/
          cpus = `nproc`.to_i
          # meminfo shows KB and we need to convert to MB
          mem = `grep 'MemTotal' /proc/meminfo | sed -e 's/MemTotal://' -e 's/ kB//'`.to_i / 1024 / 4
        end
          
        # Use VBoxManage to customize the VM. For example to change memory:
        vb.customize ["modifyvm", :id, "--memory", mem]
        vb.customize ["modifyvm", :id, "--cpus", cpus]
      end
    end

### Fixing Issue with Setting Up Host-only Network

If, **vagrant up** fails to start the VM because of some conflicting network adapters, the following workaround should fix it:

    $ VBoxManage list dhcpservers
    NetworkName:    HostInterfaceNetworking-vboxnet0
    IP:             192.168.56.100
    NetworkMask:    255.255.255.0
    lowerIPAddress: 192.168.56.101
    upperIPAddress: 192.168.56.254
    Enabled:        Yes

    $ VBoxManage dhcpserver remove --netname HostInterfaceNetworking-vboxnet0

### Creating our own Vagrant base box

First, do the usual vagrant init, up and ssh. Then install the necessary packages, and exit the SSH session.

Find the VM name as listed on VirtualBox:

    $ VBoxManage list vms

To create our own Vagrant box containing the extra installed packages:

    $ vagrant package --base <vm_name_on_virtualbox> --output <base_box_name>

Add the newly created base box to the list of base boxes:

    $ vagrant box add <box_name_to_add_to_list> <base_box_name>

### Managing the List of Vagrant boxes

    $ vagrant box <subcommand>

Sub-commands include:

* add
* list
* outdated
* remove
* repackage
* update


## References

* [Vagrant Documentation](http://docs.vagrantup.com)
* [How to make Vagrant performance not suck](http://www.stefanwrobel.com/how-to-make-vagrant-performance-not-suck)
* [dhcp private_network fails on virtualbox - Issue #3083](https://github.com/mitchellh/vagrant/issues/3083)
