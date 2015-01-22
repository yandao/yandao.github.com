---
layout: post
title: "Setup Mac for Development Purposes"
description: ""
category: configuration
tags: [osx, zsh, homebrew-cask, vagrant, virtualbox]
---
{% include JB/setup %}


## Motivation

Because a software developer needs to "power up" from time to time, by installing even better tools, or setting up existing tools the better way (i.e. using package managers).


## Pre-requisites

NOTE: Obviously already installed, but for archival purposes, they are listed below.

### Install XCode command line tools

On Mavericks, the command is:

    $ xcode-select --install

### Install Homebrew

    $ ruby -e "$(curl -fsSL https://raw.githubusercontent.com/Homebrew/install/master/install)"


## Homebrew package maintenance

Commands to run from time to time to keep the formulae "fresh":

    $ brew doctor

    $ brew update
    $ brew outdated
    $ brew upgrade <outdated_formulae>

    $ brew cleanup -s --force


## Install Homebrew Cask

For installing and managing OS X apps that are distributed as binaries. Just type on the command line instead of dragging and dropping stuff.

    $ cd $( brew --prefix )
    $ brew install caskroom/cask/brew-cask

List all available casks:

    $ brew cask search

Install a cask
    
    $ brew cask install <cask>

    NOTE: Some examples include Google Chrome, Dropbox, iTerm2, Vagrant, Virtualbox, Transmission, VLC etc.

List installed casks

    $ brew cask list


## Install zsh

Install zsh from Homebrew:

    $ brew install zsh

If OS X complains about zsh being a non-standard shell:

    $ sudo vim /etc/shells

        # List of acceptable shells for chpass(1).
        # Ftpd will not allow users to connect who are not using
        # one of these shells.
        
        ...
        /bin/zsh
        /usr/local/bin/zsh

Set zsh as default shell:

    $ chsh -s `which zsh`


## Customise zsh

I use [oh-my-zsh](https://github.com/robbyrussell/oh-my-zsh) for now.

    $ git clone git://github.com/robbyrussell/oh-my-zsh.git ~/.oh-my-zsh
    $ cp ~/.zshrc ~/.zshrc.orig
    $ cp ~/.oh-my-zsh/templates/zshrc.zsh-template ~/.zshrc

Set the [theme](https://github.com/robbyrussell/oh-my-zsh/wiki/themes) (I use muse for now):

    $ vim ~/.zshrc

        ...
        ZSH_THEME="muse"
        ...

Include whatever [plugins](https://github.com/robbyrussell/oh-my-zsh/wiki/Plugins) you fancy:

    $ vim ~/.zshrc

        ...
        plugins=(brew brew-cask git history history-substring-search jsontools mvn python ruby rvm vagrant web-search)
        ...



## Further improvements

Use [dotfiles](http://zachholman.com/2010/08/dotfiles-are-meant-to-be-forked/). The lack of clean method to revert to original settings when things *go wrong* kind of discourages me to try them out for the time being. Obviously I have made small customisations to my Vim dotfile, but nothing like a full-scale customisation to various dotfiles to level up your geek-fu.


## References

* [First steps with Mac OS X as a Developer - Carlos Alexandro Becker](http://carlosbecker.com/posts/first-steps-with-mac-os-x-as-a-developer/)
* [Setup Mac OS X Mountain Lion or Mavericks](https://gist.github.com/millermedeiros/6615994)
