---
layout: post
title: "Setup Tomcat on Mac"
description: ""
category: configuration
tags: [java, tomcat]
---
{% include JB/setup %}


## Install Tomcat

Install from Homebrew:

    $ brew install tomcat


## Initial configuration

By default, **CATALINA_BASE** is set to **/usr/local/opt/tomcat/libexec**.

Correspondingly, the startup/shutdown scripts are located at **/usr/local/opt/tomcat/libexec/bin**.

To start:

    $ cd /usr/local/opt/tomcat/libexec/bin
    $ bash startup.sh

To shutdown:

    $ cd /usr/local/opt/tomcat/libexec/bin
    $ bash shutdown.sh

To modify the default configurations, for example, to use a different JRE, create the file **setenv.sh** in the same directory as the scripts:

    $ cd /usr/local/opt/tomcat/libexec/bin
    $ vim setenv.sh

        #!/usr/bin/env bash

        JRE_HOME=/Library/Java/JavaVirtualMachines/jdk1.7.0_xx.jdk/Contents/Home
        JAVA_OPTS="-Xms128m -Xmx1024m"
        CATALINA_PID="$CATALINA_BASE/tomcat.pid"

Restart Tomcat and it should use the configurations defined in **setenv.sh**.

Finally, access [http://localhost:8080](http://localhost:8080) from the web browser to make sure Tomcat is running correctly.

## References

* [4 Ways to Change JRE for Tomcat](http://www.codejava.net/servers/tomcat/4-ways-to-change-jre-for-tomcat)
* [Tomcat configuration recommendations](https://docs.oracle.com/cd/E40518_01/integrator.311/integrator_install/src/cli_ldi_server_config.html)
