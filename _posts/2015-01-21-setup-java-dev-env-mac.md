---
layout: post
title: "Setup Java Development Environment on Mac"
description: ""
category: configuration 
tags: [java, jenv]
---
{% include JB/setup %}


## Install JDK

To install the latest version of JDK:

    $ brew cask install java

To install older versions of JDK:

    $ brew tap caskroom/versions
    $ brew cask install java7


## 02. Install jEnv

Use [jEnv](http://www.jenv.be/) to manage multiple JDK installations for various Java-based projects. To install:

    $ git clone https://github.com/gcuisinier/jenv.git ~/.jenv

Add the following lines to your default shell profile:

    $ echo 'export JENV_ROOT=/usr/local/var/jenv' >> ~/.zshrc
    $ echo 'if which jenv > /dev/null; then eval "$(jenv init -)"; fi' >> ~/.zshrc

**NOTE**:Replace zshrc with bash_profile if using bash.

Restart shell:

    $ exec $SHELL -l


## 03. Add installed JDK paths to jEnv

Replace 'xx' with the correct revision numbers.

    $ jenv add /Library/Java/JavaVirtualMachines/jdk1.7.0_xx.jdk/Contents/Home/
    $ jenv add /Library/Java/JavaVirtualMachines/jdk1.8.0_xx.jdk/Contents/Home/

Check that they are added correctly:

    $ jenv versions

        system
        1.7
        1.7.0.xx
        1.8
        1.8.0.xx
        oracle64-1.7.0.xx
        oracle64-1.8.0.xx

To set default JDK version, e.g 1.7:

    $ jenv global 1.7

To set a particular version of JDK to a project:

    $ cd <path_to_the_project>
    $ jenv local <jdk_version>


## 04. Upgrading JDK

To upgrade JDK to a newer version frm Homebrew Cask:

    $ brew cask install java --force --download

which basically force re-install the package.


