---
layout: post
title: "Remove element from List object while iterating in Java"
description: ""
category: tutorial
tags: [java]
---
{% include JB/setup %}


## The wrong way

    for (Object obj : list) {
        if (condition is true) {
            list.remove(obj);
        }
    }

This will result in 

    Exception in thread "main" java.util.ConcurrentModificationException


## The correct (or safe) way

    for (Iterator<Object> itr = list.iterator(); itr.hasNext();) {
        Object obj = itr.next();

        if (condition is true) {
            itr.remove();
        }
    }


## Reference

* [Efficient equivalent for removing elements while iterating the Collection - Stack Overflow](http://stackoverflow.com/questions/223918/efficient-equivalent-for-removing-elements-while-iterating-the-collection)