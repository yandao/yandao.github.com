---
layout: post
title: "How to Update Existing Vagrant VM"
description: ""
category: tutorial
tags: [vagrant]
---
{% include JB/setup %}


### 01. Include the following line in Vagrantfile:

    config.ssh.insert_key = false


### 02. Create a new Vagrant VM.

### 03. Do the updates/upgrades/install packages/config etc etc.

### 04. Replace the entry in .ssh/authorized_keys with the insecure vagrant.pub

    $ wget --no-check-certificate https://raw.github.com/mitchellh/vagrant/master/keys/vagrant.pub -O authorized_keys

    $ chmod 600 authorized_keys
    $ chmod 700 .ssh
    $ chown -R vagrant:vagrant .ssh

### 05. Cleaning up

    $ sudo apt-get autoremove -y
    $ sudo apt-get autoclean -y
    
    OR

    $ sudo yum clean all

    $ sudo rm -f /etc/udev/rules.d/70-persistent-net.rules
    $ sudo rm -rf /tmp/*
    $ sudo rm -f /var/log/wtmp /var/log/btmp

    $ sudo dd if=/dev/zero of=/EMPTY bs=1M
    $ sudo rm -f /EMPTY

    $ cat /dev/null > ~/.bash_history && history -c && exit

### 06. Replace the base box.

    $ vagrant package --output <base_box>.box
    $ vagrant box remove <base_box>
    $ vagrant box add <base_box> <base_box>.box

### 07. When booting up other existing VMs, they should detect the insecure public key and regenerate a new key pair.
