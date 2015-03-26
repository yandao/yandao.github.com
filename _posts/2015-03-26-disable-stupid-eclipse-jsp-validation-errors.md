---
layout: post
title: "Disable Stupid Eclipse JSP Validation Errors"
description: ""
category: quick tips
tags: [java, jsp]
---
{% include JB/setup %}


## The Fix

If you use Eclipse to develop Java web applications, you may encounter a seemingly serious JSP validation error similar to the following:

    An error occurred at line: 459 in the generated java file
    Duplicate local variable configurationExists

    An error occurred at line: 576 in the generated java file
    Duplicate local variable configurationExists

    An error occurred at line: 693 in the generated java file
    Duplicate local variable configurationExists

    ...

Based on online discussions, it appears that JSP validation in Eclipse is actually throwing an error for a perfectly valid JSP syntax. So the recommended fix is to simply disable the validation check on Eclipse.

1. Go to "Project->Properties->Validation".
2. Click "Configure Workspace Settings...".
3. Unselect options for JSP Syntax Validator. You need to uncheck both manual and build.


## References

* [Mysterious Eclipse JSP Validation Errors](http://stackoverflow.com/a/3790176)
* [Struts 1.2 and bean:define tag throws jsp 1.1 regression exception](https://bz.apache.org/bugzilla/show_bug.cgi?id=48616)