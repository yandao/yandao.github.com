---
layout: post
title: "How to Change Screen Resolution of OS X Guest (VirtualBox)"
description: ""
category: quick tips
tags: [osx, virtualbox]
---
{% include JB/setup %}


From the official [user manual](https://www.virtualbox.org/manual/ch03.html#efividmode):

    3.12.1. Video modes in EFI

    EFI provides two distinct video interfaces: GOP (Graphics Output Protocol) and UGA (Universal Graphics Adapter). Mac OS X uses GOP, while Linux tends to use UGA. VirtualBox provides a configuration option to control the framebuffer size for both interfaces.

    To control GOP, use the following VBoxManage command:

    $ VBoxManage setextradata "VM name" VBoxInternal2/EfiGopMode N
    
    Where N can be one of 0,1,2,3,4,5 referring to the 640x480, 800x600, 1024x768, 1280x1024, 1440x900, 1920x1200 screen resolution respectively.

