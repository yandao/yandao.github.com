---
layout: post
title: "Create init script on Ubuntu"
description: ""
category: quick tips
tags: [ubuntu]
---
{% include JB/setup %}


## The steps

Create a shell script file at /etc/init.d/&lt;script_file&gt;

    #!/usr/bin/env bash

    ### BEGIN INIT INFO
    # Provides:          <service_name>
    # Required-Start:    $remote_fs $syslog
    # Required-Stop:     $remote_fs $syslog
    # Default-Start:     2 3 4 5
    # Default-Stop:      0 1 6
    # Short-Description: <short_description>
    # Description:       <full_description>
    ### END INIT INFO

    case "$1" in
    start)
    log_action_begin_msg "Starting service..."
    <command_to_start_service>
    ;;
    stop)
    log_action_begin_msg "Stopping service..."
    <command_to_stop_service>
    ;;
    restart)
    $0 stop
    $0 start
    ;;
    esac
    exit 0
    EOT

Make the script executable

    $ sudo chmod +x /etc/init.d/<script_file>

Configure it to start on boot

    $ sudo sysv-rc-conf <script_file> on


## References

* [Startup script on Ubuntu 12.04 not getting executed. Dependencies / load order.](http://superuser.com/a/556122)
* [Why is chkconfig no longer available in Ubuntu?](http://askubuntu.com/a/277732)
