---
layout: post
title: "Manage Ruby Development Environment using rbenv on Mac"
description: ""
category: configuration 
tags: [ruby, rbenv]
---
{% include JB/setup %}


## 01. Install rbenv

Use [rbenv](https://github.com/sstephenson/rbenv) to manage multiple Ruby binaries for various Ruby-based projects. To install:

    $ brew install rbenv ruby-build rbenv-gem-rehash

Add the following lines to your default shell profile:

    $ echo 'export RBENV_ROOT=/usr/local/var/rbenv' >> ~/.zshrc
    $ echo 'if which rbenv > /dev/null; then eval "$(rbenv init -)"; fi' >> ~/.zshrc

**NOTE**:Replace zshrc with bash_profile if using bash.

Restart shell:

    $ exec $SHELL -l


## 02. Install Ruby binaries

To list all available Ruby binaries:

    $ rbenv install -l

To install a Ruby binary, e.g. 2.2.0:

    $ rbenv install 2.2.0

Check that the binaries are installed correctly:

    $ rbenv versions

        system
        2.2.0

To set default Ruby version, e.g 2.2.0:

    $ rbenv global 2.2.0

To set a particular version of Ruby to a project:

    $ cd <path_to_the_project>
    $ rbenv local <ruby_version>

Update to the latest Rubygems version:

    $ gem update --system

Install the bundler gem:

    $ gem install bundler

Check that bundler is installed:

    $ bundler -v

If system complains that bundler is not found, start a new shell.


## 03. Using Bundler to manage different sets of gems

Instead of installing all gems in one global list for all projects, it may be a good practice to separate them into sets local to a particular Ruby-based project. [Bundler](http://bundler.io/) allows us to manage the list of required gems for the project.

Firstly, configure Bundler with the following settings:

    $ bundle config --global bin bin
    $ bundle config --global path .bundle

The first command configures Bundler to generate binstubs in the bin directory of the application. Executable gems are then called as ./bin/<exec_gem> instead of bundle exec <exec_gem>.

The second command tells Bundler to store the installed gems in the app's .bundle directory. This effectively causes each project with a Gemfile to be treated as an implicit gemset.


## 04. Define Ruby gems to install

The required gems for a project is typically defined in a file called **Gemfile**. If a Gemfile is not defined yet, create one by running:

    $ bundle init

The default content looks like the following:

    # A sample Gemfile
    source "https://rubygems.org"

    # gem "rails"

Add the required gems, e.g.:

    # A sample Gemfile
    source "https://rubygems.org"

    gem 'nokogiri'
    gem 'rails'

Install the required gems using Bundler:

    $ bundle install

which will install the gems in the .bundle directory as defined in the configuration above.


## References

* [Using rbenv to Manage Rubies and Gems](http://robots.thoughtbot.com/using-rbenv-to-manage-rubies-and-gems)
* [Implicit Gemsets with rbenv](http://devoh.com/blog/2012/07/implicit-gemsets-with-rbenv)

