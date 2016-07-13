---
layout: post
title: "PHP 5.6 + Nginx 1.10.1 + PostgreSQL 9.5.3 on Ubuntu Trusty"
description: ""
category: configuration
tags: []
---
{% include JB/setup %}


## Pre-requisite

Set LC_ALL to any UTF-8 locale.

    $ vim /etc/default/locale

        LC_ALL="en_US.UTF-8"

## Add Repositories

PHP

    $ sudo add-apt-repository ppa:ondrej/php

Nginx

    $ sudo add-apt-repository ppa:nginx/stable

PostgreSQL

    $ sudo sh -c 'echo "deb http://apt.postgresql.org/pub/repos/apt/ $(lsb_release -cs)-pgdg main" > /etc/apt/sources.list.d/pgdg.list'

    $ wget --quiet -O - https://www.postgresql.org/media/keys/ACCC4CF8.asc | sudo apt-key add -

Update all of them

    $ sudo apt-get update


## Install Packages

PHP (base and essential packages)

    $ sudo apt-get install php5.6 php5.6-common php5.6-fpm php5.6-json php5.6-mcrypt php5.6-mbstring php5.6-pgsql

Nginx

    $ sudo apt-get install nginx

PostgreSQL

    $ sudo apt-get install postgresql-9.5 postgresql-contrib-9.5


## Initial Configurations

PHP-FPM

    $ sudo vim /etc/php/5.6/fpm/pool.d/www.conf

        user = www-data
        group = www-data

        listen = /run/php/php5.6-fpm.sock
        listen.user = www-data
        listen.group = www-data

Self-signed certificate

    $ sudo openssl req -x509 -nodes -days <num_days> -newkey rsa:2048 -keyout /path/to/ssl.key -out /path/to/ssl.crt

Nginx

    $ sudo vim /etc/nginx/sites-available/default

        server {
            listen 80 default_server;
            listen [::]:80 default_server;
            listen 443 ssl;

            root /var/www;

            # Add index.php to the list if you are using PHP
            index index.php index.html index.htm index.nginx-debian.html;

            server_name _;

            ssl_certificate /path/to/ssl.crt;
            ssl_certificate_key /path/to/ssl.key;

            location / {
                try_files $uri $uri/ /index.php?$query_string;
            }

            location ~ \.php$ {
                try_files $uri /index.php =404;
                fastcgi_split_path_info ^(.+\.php)(/.+)$;
                fastcgi_pass unix:/var/run/php/php5.6-fpm.sock;
                fastcgi_index index.php;
                fastcgi_param SCRIPT_FILENAME $document_root$fastcgi_script_name;
                include fastcgi_params;
            }
        }

PostgreSQL

    $ sudo su - postgres
    $ psql

        postgres=# CREATE USER <db_user> WITH PASSWORD '<password>';
        postgres=# CREATE DATABASE <db_name>;
        postgres=# GRANT ALL PRIVILEGES ON DATABASE <db_name> TO <db_user>;
        postgres=# ALTER DATABASE <db_name> OWNER TO <db_user>;
        postgres=# \q

    $ exit

    $ sudo vim /etc/postgresql/9.5/main/postgresql.conf

        listen_addresses = 'localhost' // comma-separated, '*' for all

        ...

        ssl_cert_file = '/path/to/ssl.crt'
        ssl_key_file = '/path/to/ssl.key'


    $ sudo vim /etc/postgresql/9.5/main/pg_hba.conf

        # TYPE  DATABASE        USER            ADDRESS                 METHOD
        host    all             all             127.0.0.1/24            md5

phpPgAdmin (download the standalone tarball)

    $ tar jxvf phpPgAdmin-x.x.tar.bz2
    $ rm phpPgAdmin-x.x.tar.bz2

    $ sudo mv -v phpPgAdmin-x.x /var/www/phpPgAdmin
    $ sudo chown -R www-data:www-data /var/www/phpPgAdmin

    $ sudo vim /var/www/phpPgAdmin/conf/config.inc.php

        $conf['servers'][0]['desc'] = 'Host Name';
        $conf['servers'][0]['host'] = 'URL/IP Address';
        $conf['servers'][0]['port'] = 5432;

        ...

        $conf['extra_login_security'] = true;

        // other means. (e.g. Run 'SELECT * FROM pg_database' in the SQL area.)
        $conf['owned_only'] = true;

        ...

Restart PostgreSQL, PHP-FPM and Nginx

    $ sudo service postgresql restart
    $ sudo service php5.6-fpm restart
    $ sudo service nginx restart

If everything goes well, phpPgAdmin should be accessible at http://&lt;host_url&gt;/phpPgAdmin
