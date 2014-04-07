---
layout: post
title: "Fix the \"Failure Initializing Default System SSL Context\" Issue"
description: ""
category: 
tags: [java, gradle]
---
{% include JB/setup %}


## Introduction

After upgrading to OS X Mavericks (finally!), it is expected that some stuff becomes non-functional as some files get moved/removed. One such issue is the cacerts file in the default Java installation goes missing, which is required when the Gradle tool tries to resolve project dependencies, and hence needs to be replaced. The following script does the job (need to run as root):

    #!/usr/bin/env bash

    set -e
    # The location of certificate file which gradle asks for, create the directory if it does not exist.
    cacerts=/System/Library/Java/Support/CoreDeploy.bundle/Contents/Home/lib/security/cacerts

    curl 'https://docs.codehaus.org/download/attachments/158859410/startssl-CA.pem?version=1&modificationDate=1277952972158' | keytool -import -alias StartSSL-CA -file /dev/stdin -keystore $cacerts -storepass changeit -noprompt

    curl 'https://docs.codehaus.org/download/attachments/158859410/startssl-Intermediate.pem?version=1&modificationDate=1277952972182' | keytool -import -alias StartSSL-Intermediate -file /dev/stdin -keystore $cacerts -storepass changeit -noprompt


## Reference

* [Fix the "Failure Initializing Default System SSL Context" Issue](http://fedcuit.github.io/blog/2013/12/30/fix-jdk-ssl-certificate-issue)