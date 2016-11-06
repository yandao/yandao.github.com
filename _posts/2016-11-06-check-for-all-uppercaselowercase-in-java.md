---
layout: post
title: "Check for All Uppercase/Lowercase in Java"
description: ""
category: quick tips
tags: [java]
---
{% include JB/setup %}


## Using Standard JDK

    public static boolean isUpperCase(String s) {
        for (int i = 0; i < s.length(); i++) {
            if (Character.isLowerCase(s.charAt(i)))
                return false;
        }

        return true;
    }


**NOTE**: There are also alternatives using Apache Commons or Guava libraries, but they may run into issues when the string contains punctuations/other symbols.


## Reference

* [Is there an existing library method that checks if a String is all upper case or lower case in Java? - Stack Overflow](http://stackoverflow.com/a/677592)