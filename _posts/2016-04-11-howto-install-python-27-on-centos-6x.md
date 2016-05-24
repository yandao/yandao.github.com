---
layout: post
title: "How to Install Python 2.7 on CentOS 6.x"
description: ""
category: tutorial
tags: [python, centos]
---
{% include JB/setup %}



## Background

CentOS 6.x included Python up to version 2.6.x only, which has been deprecated since October 2013.

Additionally, some packages require Python 2.7.x or newer, therefore it is necessary to install Python 2.7.x from some external source. The steps are described below.


## Install Python 2.7 on CentOS 6.x

    # Add the SCL repo
    $ sudo yum install -y centos-release-SCL

    # Install Python 2.7
    $ sudo yum install -y python27


## Install Pip with Python 2.7 on CentOS 6.x

    # Since the system defaults to 2.6, it is necessary to enable Python 2.7
    $ . /opt/rh/python27/enable
    $ cd /opt/rh/python27/root/usr/bin/

    # Install Pip
    $ sudo LD_LIBRARY_PATH=$LD_LIBRARY_PATH ./easy_install-2.7 pip


## Install Python packages using Python 2.7 on CentOS 6.x

    # Make sure Python 2.7 is enabled as described above
    $ sudo LD_LIBRARY_PATH=$LD_LIBRARY_PATH ./pip2.7 install <package_name>


## References

* [[Python-Dev] Python 2.6 to end support with 2.6.9 in October 2013](https://mail.python.org/pipermail/python-dev/2013-September/128287.html)
* [Redhat / CentOS 6.x users need python 2.7](https://community.letsencrypt.org/t/redhat-centos-6-x-users-need-python-2-7/2190)
* [The Software Collections ( SCL ) Repository](https://wiki.centos.org/AdditionalResources/Repositories/SCL)