---
layout: post
title: "Setup Apache + PHP on Mac"
description: ""
category: configuration 
tags: [apache, php]
---
{% include JB/setup %}


**NOTE**: For this tutorial, the installed Apache version is 2.4.10 and the installed PHP version is 5.4.36.


## 01. Install Apache 2

Firstly, terminate any running httpd processes:

    $ ps auwxx | grep httpd
    $ kill -HUP <pid>

Then proceed to install Apache 2:

    $ brew tap homebrew/dupes
    $ brew tap homebrew/apache
    $ brew tap httpd24 --with-brewed-openssl

Check that it is installed correctly:

    $ which httpd

        /usr/local/bin/httpd

Make sure that the web server can run on its default configuration:

    $ sudo apachectl start

Enter the URL [http://localhost:8080](http://localhost:8080) on the web browser and you should see the default index.html page.


## 02. Install PHP 5.4 

Initialise from Homebrew:

    $ brew tap homebrew/homebrew-php
    $ brew search php54

List available install options (since the web server is Apache, --with-apache should be included):

    $ brew options php54
    
Install from Homebrew:

    $ brew install php54 --with-apache --homebrew-apxs

Ensure that PHP is installed correctly:

    $ php -v


## 03. Modify httpd.conf

    $ vim /usr/local/etc/apache2/2.4/httpd.conf

        ...
        LoadModule php5_module /usr/local/opt/php54/libexec/apache2/libphp5.so

        ...
        
        <Directory ...
            AllowOverride All

        ...

        <IfModule dir_module>
            DirectoryIndex index.php index.html

        ...

        AddType application/x-httpd-php .php

        ...

        # probably not needed
        AddHandler application/x-httpd-php .php


## 04. Create test PHP page

    $ echo '<?php phpinfo();' > /usr/local/var/www/htdocs/index.php


## 05. Restart Apache

    $ sudo apachectl restart


## 06. Load test PHP page

Enter the URL [http://localhost:8080/index.php](http://localhost:8080/index.php) on the web browser. If the info page loads correctly, then the setup is correct.


## Reference

* [Macでapacheを2.2から2.4にアップデートする(挫折したので2.2に戻した)](http://qiita.com/awm-kaeruko/items/624146f4c86cf1dd42ed)
