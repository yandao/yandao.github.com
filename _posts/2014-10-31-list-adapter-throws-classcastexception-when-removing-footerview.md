---
layout: post
title: "List Adapter throws ClassCastException when removing FooterView"
description: ""
category: tutorial
tags: [android]
---
{% include JB/setup %}


## Explanation

From the reference below:

    Call this before calling listView.setAdapter(adapter). This is so ListView can wrap the supplied cursor with one that will also account for header and footer views.

Good practice: 

Instantiate the adapter early, e.g. during onCreate() in Activity or onActivityCreated() in Fragment, and immediately set the list view to use it. Since it is most likely still empty, once you have added items to the adapter and need to refresh the contents of the list view, simply call **adapter.notifyDataSetChanged()**.


## Reference

* [Adapter class cast exception when removing a Footer view? - Stack Overflow](http://stackoverflow.com/questions/12649423/adapter-class-cast-exception-when-removing-a-footer-view)