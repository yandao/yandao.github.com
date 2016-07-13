---
layout: post
title: "Some Useful Jenkins REST API endpoints"
description: ""
category: quick tips
tags: [jenkins]
---
{% include JB/setup %}


## Introduction

Somehow it is just impossible to find a complete reference to all available API endpoints on Jenkins. This post attempts to list a few that may be useful (at least for myself).

The list below uses the [cURL](https://curl.haxx.se/) command for quick and simple operations. It also assumes that Jenkins server does not allow anonymous access.


## Get Build Status

Specific build number

    $ curl -s http://<jenkins_url>/job/<job_name>/<specific_build_number>/api/json --user <user_name>:<api_token>

Last build

    $ curl -s http://<jenkins_url>/job/<job_name>/lastBuild/api/json --user <user_name>:<api_token>

Last successful build

    $ curl -s http://<jenkins_url>/job/<job_name>/lastSuccessfulBuild/api/json --user <user_name>:<api_token>

Last failed build

    $ curl -s http://<jenkins_url>/job/<job_name>/lastFailedBuild/api/json --user <user_name>:<api_token>

Filter only desired output, e.g. build id, duration etc

    $ curl -s http://<jenkins_url>/job/<job_name>/lastBuild/api/json\?tree\=number,building,result,timestamp --user <user_name>:<api_token>


## Administer a Build

**NOTE**: To prevent CSRF, Jenkins require POST requests to include a crumb, which is specific to each user. The command to obtain the crumb is:

    $ curl http://<jenkins_url>/crumbIssuer/api/xml\?xpath\=concat\(//crumbRequestField,%22:%22,//crumb\) --user <user_name>:<api_token>

Start a build

    $ curl -H ".crumb:<crumb_string>" -X POST http://<jenkins_url>/job/<job_name>/build --user <user_name>:<api_token>

Start a parameterised build

    $ curl -H ".crumb:<crumb_string>" -X POST http://<jenkins_url>/job/<job_name>/buildWithParameters --data-urlencode json='{"parameter":[{"<key>":"<value>"}]}' --user <user_name>:<api_token>

Stop a build (need specific build number)

    $ curl -H ".crumb:<crumb_string>" -X POST http://<jenkins_url>/job/<job_name>/<build_number>/stop --user <user_name>:<api_token>

Get scheduled build(s) currently in queue

    $ curl -s http://<jenkins_url>/queue/api/json --user <user_name>:<api_token>

Cancel scheduled build from the queue

    $ curl -H ".crumb:<crumb_string>" -X POST http://<jenkins_url>/queue/cancelItem\?id\=<queue_number> --user <user_name>:<api_token>
