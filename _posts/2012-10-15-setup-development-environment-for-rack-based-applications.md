---
layout: post
title: "Setup Development Environment for Rack-based applications"
description: ""
category: configuration
tags: [nginx, unicorn, ruby, rack]
---
{% include JB/setup %}


## Order of Installation

* Nginx
* RVM
* Ruby
* Unicorn 

plus their dependencies. 

## Install Nginx

### Mac OS X

Just use [Homebrew](https://github.com/mxcl/homebrew/wiki/Installation)... seriously.

    $ brew install nginx

### Ubuntu Linux

    $ sudo apt-get install nginx


## Install RVM

    $ curl -L get.rvm.io | bash -s stable
    $ source ~/.rvm/scripts/rvm

If cURL complains about SSL certification error, download the following certificate:

    $ wget http://curl.haxx.se/ca/cacert.pem
    $ export CURL_CA_BUNDLE=~/cacert.pem

To install a version of Ruby, say 1.9.3:

    $ rvm install 1.9.3

_NOTE_: There may be [some issues](http://stackoverflow.com/questions/9434002/how-to-solve-ruby-installation-is-missing-psych-error) with installing version 1.9.3 due to the change in the default YAML parser. I encountered this in my CentOS 5.7 setup and couldn't make it work despite trying all the different solutions provided by other users. Ended up using 1.8.7 instead.

_UPDATE (22 Oct 2012)_: Found the fix!

    $ rvm pkg install libyaml
    $ rvm pkg install readline

    $ rvm install 1.9.3 --with-readline-dir=~/.rvm/usr --with-libyaml-dir=~/.rvm/usr

To set a version as default:

    $ rvm use <version> --default

To load RVM into shell session permanently, put the following at the end of the bash profile (/.bashrc):

    $ vim ~/.bashrc

        ...

        [[ -s "$HOME/.rvm/scripts/rvm" ]] && . "$HOME/.rvm/scripts/rvm"

### (Optional) Create Project-Specific Gemsets

RVM allows for installations of multiple Ruby versions on the same machine, with a set of gems installed in default locations for each version.

RVM also allows for each individual version to have multiple gemsets applied to it, as required by different projects, for example.

To create and switch to a newly created gemset, e.g. a gemset called rails3 running on Ruby 1.8.7:

    $ rvm use 1.8.7@rails3 --create

Now we can proceed to install gems in this new gemset. For example, if we want to test Rails 3.1.3 on Ruby 1.8.7:

    $ gem install rails --version 3.1.3

To see the gems installed in this gemset:

    $ gem list --local

It is also possible to copy gemsets:

    $ rvm gemset copy <version1>@<gemset1> <version2>@<gemset2>

Read [RVM Best Practices](https://rvm.io/rvm/best-practices/) to manage each project better.

## Initial Configuration

For simplicity, we will create a "Hello World" application on Sinatra, but this should work on any Rack-based setup.

    $ gem install sinatra
    $ sudo mkdir -p /var/www/helloworld
    $ sudo vim /var/www/helloworld/app.rb

        require 'sinatra'

        get '/' do
            "Hello! The time now is #{Time.now}."
        end

Might be good to create a new group called "web" for handling all these web-based files:

    $ sudo groupadd web
    $ sudo usermod -a -G web nginx
    $ sudo chown nginx:web -R web /var/www
    $ sudo chmod -R 775 /var/www

Add yourself to group "web":

    $ sudo usermod -a -G web <username>

Ensure that port 80 (or whichever port you want to use in Nginx) is open:

    $ sudo cat /etc/sysconfig/iptables

        ...

        -A RH-Firewall-1-INPUT -m state --state NEW -m tcp -p tcp --dport 80 -j ACCEPT


## Setup config.ru

    $ sudo vim /var/www/helloworld/config.ru

        require 'sinatra'

        set :env,  :production
        disable :run

        require './app.rb'

        run Sinatra::Application


## Setup Unicorn configuration file

Please refer to links in [References](#references) for better understanding on what each line is doing.

    $ sudo vim unicorn.config

        worker_processes 4
        working_directory "/var/www/helloworld"
        listen 'unix:/tmp/unicorn.sock', :backlog => 512
        timeout 120
        pid "/tmp/basic_unicorn.pid"

        # This loads the application in the master process before forking worker procs
        preload_app true
        if GC.respond_to?(:copy_on_write_friendly=)
            GC.copy_on_write_friendly = true
        end

        ##
        # When sent a USR2, Unicorn will suffix its pidfile with .oldbin and
        # immediately start loading up a new version of itself (loaded with a new
        # version of our app). When this new Unicorn is completely loaded
        # it will begin spawning workers. The first worker spawned will check to
        # see if an .oldbin pidfile exists. If so, this means we've just booted up
        # a new Unicorn and need to tell the old one that it can now die. To do so
        # we send it a QUIT.
        #
        # Using this method we get 0 downtime deploys.
        before_fork do |server, worker|
            old_pid = "#{server.config[:pid]}.oldbin"
            if File.exists?(old_pid) && server.pid != old_pid
                begin
                    Process.kill("QUIT", File.read(old_pid).to_i)
                    rescue Errno::ENOENT, Errno::ESRCH
                    # someone else did our job for us
                end
            end
        end

        # Set the path of the log files inside the log folder of the testapp
        stderr_path "/path/to/unicorn.stderr.log"
        stdout_path "/path/to/unicorn.stdout.log"


## Deploy Rack application

Now we can deploy our "Hello World" Sinatra application on Unicorn:

    $ unicorn -c unicorn.config &

In this case, the "Hello World" application is listening to the socket **/tmp/unicorn.sock** as defined in unicorn.config above. The next step is to "link" it in our Nginx configuration.


## Setup Nginx configuration file

    $ sudo vim /etc/nginx/nginx.conf

        events {
            worker_connections  1024;
            accept_mutex off;
            use epoll;
        }

        ...

        upstream app_server {
            server unix:/tmp/unicorn.sock fail_timeout=0;
        }

        server {
            listen 80 default deferred;

            client_max_body_size 4G;
            server_name _;

            keepalive_timeout 5;

            root /var/www/helloworld;

            try_files $uri/index.html $uri.html $uri @app;

            location @app {
                proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
                proxy_set_header Host $http_host;
                proxy_redirect off;
                proxy_pass http://app_server;
            }
        }

The "link" to the "Hello World" application deployed on Unicorn is defined in the **upstream** part. The socket file must match the one defined in unicorn.config.

Restart Nginx:

    $ sudo service nginx restart

Ensure that the socket is created:

    $ sudo netstat -nalx | grep unicorn.sock

Ensure that port 80 (or whichever port you want to use in Nginx) is open and receiving requests:

    $ sudo netstat - nalt | grep :80

If everything is setup correctly, any HTTP requests to http://&lt;domain_name&gt; will be redirected to our "Hello World" application.


## References

* [Unicorn! - The GitHub Blog](https://github.com/blog/517-unicorn)
* [Nginx + Unicorn sample configuration](https://github.com/defunkt/unicorn/blob/master/examples/nginx.conf)
* [Setting up Unicorn with Nginx](http://sirupsen.com/setting-up-unicorn-with-nginx/)
* [Running Sinatra (and other Rack apps) on Nginx + Unicorn](http://velenux.wordpress.com/2012/01/10/running-sinatra-and-other-rack-apps-on-nginx-unicorn/)
