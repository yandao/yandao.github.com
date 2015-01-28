---
layout: post
title: "Setup OCI to connect to Oracle DB from PHP on Mac"
description: ""
category: configuration
tags: [php, oracle]
---
{% include JB/setup %}


## Pre-requisites

01. Setup Apache and PHP as outlined [here](/configuration/2015/01/21/setup-apache-php-mac/).
02. An Oracle DB server setup somewhere.

**NOTE**: For this tutorial, the installed Apache version is 2.4.10 and the installed PHP version is 5.4.36.


## 01. Install Oracle Instant Client

Download the following packages from the [Oracle website](http://www.oracle.com/technetwork/topics/intel-macsoft-096467.html) (requires registration):

* Instant Client Package - Basic
* Instant Client Package - SQL*Plus
* Instant Client Package - SDK

Extract them all to the same directory, e.g.: $HOME/opt/instantclient/xx.x.x.x

    .
    ├── BASIC_README
    ├── SQLPLUS_README
    ├── adrci
    ├── genezi
    ├── glogin.sql
    ├── libclntsh.dylib.11.1
    ├── libnnz11.dylib
    ├── libocci.dylib.11.1
    ├── libociei.dylib
    ├── libocijdbc11.dylib
    ├── libsqlplus.dylib
    ├── libsqlplusic.dylib
    ├── ojdbc5.jar
    ├── ojdbc6.jar
    ├── sdk
    │   ├── SDK_README
    │   ├── demo
    │   ├── include
    │   ├── ott
    │   └── ottclasses.zip
    ├── sqlplus
    ├── uidrvci
    └── xstreams.jar


## 02. Create symbolic links

Assuming Instant Client is installed at $HOME/opt/instantclient/xx.x.x.x, create the following symlinks:

    $ ln -s /usr/local/opt/instantclient $HOME/opt/instantclient/xx.x.x.x
    $ ln -s /usr/local/opt/instantclient/sdk/include/*.h /usr/local/include/
    $ ln -s /usr/local/opt/instantclient/sqlplus /usr/local/bin/
    $ ln -s /usr/local/opt/instantclient/*.dylib /usr/local/lib/
    $ ln -s /usr/local/opt/instantclient/*.dylib.11.1 /usr/local/lib/
    $ ln -s /usr/local/lib/libclntsh.dylib.11.1 /usr/local/lib/libclntsh.dylib


## 03. Test connect to Oracle DB

Test connect to Oracle DB using SQL*Plus:

    $ sqlplus <username>@"(DESCRIPTION=(ADDRESS=(PROTOCOL=TCP)(HOST=<URL>)(PORT=1521))(CONNECT_DATA=(SID=<sid>)))"

when prompted, enter the password.


## 04. Install OCI extension with PECL

    $ pecl install oci8

If the script prompts you to provide the path to ORACLE_HOME directory, respond with:

    instantclient,/usr/local/lib

If PECL throws the following error:

    Fatal error: Call to a member function getFilelist() on a non-object in /usr/local/Cellar/php5x/x.x.xx/lib/php/PEAR/Command/Install.php on line xxx

fix with:

    $ touch $(brew --prefix php5x)/lib/php/.lock && chmod 0644 $(brew --prefix php5x)/lib/php/.lock

as described [here](https://github.com/Homebrew/homebrew-php/issues/1039#issuecomment-41307694).

Check that OCI8 extension is installed correctly:

    $ pecl list


## 05. Create test PHP page to connect to Oracle DB

    $ vim /path/to/webserver/document/root/test-oci.php

        <?php
        // Create connection to Oracle
        $conn = oci_connect("username", "password", "(DESCRIPTION=(ADDRESS=(PROTOCOL=TCP)(HOST=<URL>)(PORT=1521))(CONNECT_DATA=(SID=<sid>)))");
        if (!$conn) {
            $m = oci_error();
            echo $m['message'], "\n";
            exit;
        } else {
            print "Connected to Oracle!";
        }

        // Parse SQL statement. Note there is no final semi-colon in the SQL statement.
        $stid = oci_parse($conn, '<sql_query>');
        oci_execute($stid);

        echo "<table border='1'>\n";
        while ($row = oci_fetch_array($stid, OCI_ASSOC+OCI_RETURN_NULLS)) {
            echo "<tr>\n";
            foreach ($row as $item) {
                echo "    <td>" . ($item !== null ? htmlentities($item, ENT_QUOTES) : "&nbsp;") . "</td>\n";
            }
            echo "</tr>\n";
        }
        echo "</table>\n";

        // Close the Oracle connection
        oci_close($conn);
        ?>%


Load [http://localhost:8080/test-oci.php](http://localhost:8080/test-oci.php) and the SQL query results should be displayed on a table.


## References

* [Install PHP Oracle OCI Extension (11.2) on Mac OS X (10.8)](http://antistatique.net/blog/2013/03/25/install-php-oracle-oci-extension-11-2-on-mac-os-x-10-8/)
* [Instant Client Downloads for Mac OS X (Intel x86)](http://www.oracle.com/technetwork/topics/intel-macsoft-096467.html)
* [Connect with SQL*Plus using IP + port + SID](https://community.oracle.com/thread/2140677)
* [Using PHP with Oracle Database 11g](http://www.oracle.com/webfolder/technetwork/tutorials/obe/db/oow10/php_db/php_db.htm)
* [Oracle Database Connection Strings in PHP](https://blogs.oracle.com/alison/entry/oracle_database_connection_str)
* [PHP: oci_parse - Manual](http://php.net/manual/en/function.oci-parse.php)