---
layout: post
title: "Download specific fields of download/install stats from Google Play Developer Console"
description: ""
category: quick tips
tags: [android]
---
{% include JB/setup %}


Google provides statistics on the number of downloads/installs/uninstalls for your Android apps, which you can view on Google Play Developer Console. They also allow you to export those numbers in a CSV file at a push of a button.

Sometimes though, you may want to select only some fields that you are interested in, and be able to specify a date range too. All of them are actually defined as query strings in the following URL format:

    https://play.google.com/apps/publish/statistics/download?package=<your_app_package_name>&sd=<start_yyyymmdd>&ed=<end_yyyymmdd>&dim=overall&met=current_device_installs,daily_device_installs,daily_device_uninstalls,total_user_installs,daily_user_uninstalls

NOTE: Obviously, you need to be already logged in to access the information. Otherwise, you may wish to come up with a secure way to pass your login credentials, so that you can write a script to automate the process.


## References

* [Getting statistics from Google Play Developers with an API -- Stack Overflow](http://stackoverflow.com/a/21834017)