---
layout: post
title: "PHP 5.6 + Nginx 1.10.1 + PostgreSQL 9.5.3 on CentOS 6.8"
description: ""
category: configuration
tags: []
---
{% include JB/setup %}


## Pre-requisite

Set LC_ALL to any UTF-8 locale.

    $ sudo vim /etc/sysconfig/i18n

        LC_ALL="en_US.UTF-8"

    $ source /etc/sysconfig/i18n


## Add Repositories

EPEL

    $ sudo rpm -Uvh https://dl.fedoraproject.org/pub/epel/epel-release-latest-6.noarch.rpm

PHP

    $ sudo rpm -Uvh https://mirror.webtatic.com/yum/el6/latest.rpm

Nginx

    $ sudo rpm -Uvh http://nginx.org/packages/centos/6/noarch/RPMS/nginx-release-centos-6-0.el6.ngx.noarch.rpm

PostgreSQL (NOTE: the link says 9.5.2, but the installed version will be 9.5.3)

    $ sudo rpm -Uvh https://download.postgresql.org/pub/repos/yum/9.5/redhat/rhel-6-x86_64/pgdg-centos95-9.5-2.noarch.rpm


## Install Packages

PHP (base and essential packages)

    $ sudo yum install php56w php56w-common php56w-fpm php56w-mbstring php56w-mcrypt php56w-pear php56w-pgsql

Nginx

    $ sudo yum install nginx

PostgreSQL

    $ sudo yum install postgresql95-server postgresql95-contrib


## Initial Configurations

SELinux (allow web applications to talk to databases)

    $ sudo setsebool -P httpd_can_network_connect_db 1
    $ sudo setsebool -P httpd_can_network_connect 1

PHP-FPM

    $ sudo chkconfig php-fpm on
    $ sudo vim /etc/php-fpm.d/www.conf

        user = nginx
        group = nginx

        listen = /var/run/php-fpm/php-fpm.sock
        listen.user = nginx
        listen.group = nginx

Self-signed certificate

    $ sudo openssl req -x509 -nodes -days <num_days> -newkey rsa:2048 -keyout /path/to/ssl.key -out /path/to/ssl.crt

Nginx

    $ sudo chkconfig nginx on
    $ sudo vim /etc/nginx/conf.d/default.conf

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
                fastcgi_pass unix:/var/run/php-fpm/php-fpm.sock;
                fastcgi_index index.php;
                fastcgi_param SCRIPT_FILENAME $document_root$fastcgi_script_name;
                include fastcgi_params;
            }
        }

PostgreSQL

    $ sudo chkconfig postgresql-9.5 on
    $ sudo service postgresql-9.5 initdb
    $ sudo service postgresql-9.5 restart

    $ sudo su - postgres
    $ createuser -ePE <db_user>
    $ createdb -e -E UTF-8 -O <db_user> <db_name>
    $ exit

    $ sudo vim /var/lib/pgsql/9.5/data/postgresql.conf

        listen_addresses = 'localhost' // comma-separated, '*' for all

        ...

        ssl = on
        ssl_cert_file = '/path/to/ssl.crt'
        ssl_key_file = '/path/to/ssl.key'

    $ sudo vim /var/lib/pgsql/9.5/data/pg_hba.conf

        # TYPE  DATABASE        USER            ADDRESS                 METHOD
        host    all             all             127.0.0.1/24            md5

phpPgAdmin (download the standalone tarball)

    $ tar jxvf phpPgAdmin-x.x.tar.bz2
    $ rm phpPgAdmin-x.x.tar.bz2

    $ sudo mv -v phpPgAdmin-x.x /var/www/phpPgAdmin
    $ sudo chown -R nginx:nginx /var/www/phpPgAdmin

    $ sudo vim /var/www/phpPgAdmin/conf/config.inc.php

        $conf['servers'][0]['desc'] = 'Host Name';
        $conf['servers'][0]['host'] = 'URL/IP Address';
        $conf['servers'][0]['port'] = 5432;

        ...

        $conf['extra_login_security'] = true;

        // other means. (e.g. Run 'SELECT * FROM pg_database' in the SQL area.)
        $conf['owned_only'] = true;

        ...

    $ sudo restorecon -R -v /var/www/phpPgAdmin

Restart PostgreSQL, PHP-FPM and Nginx

    $ sudo service postgresql-9.5 restart
    $ sudo service php-fpm restart
    $ sudo service nginx restart

If everything goes well, phpPgAdmin should be accessible at https://&lt;host_url&gt;/phpPgAdmin, and you should be able to login using the DB user name created above and have full access to the database created above.

