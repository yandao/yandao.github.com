---
layout: post
title: "Convert Images to PDF with ImageMagick"
description: ""
category: quick tips
tags: [imagemagick]
---
{% include JB/setup %}


## The command

The following command will convert a series of image files into a PDF file that fit into A4 paper size.

    $ convert a.png b.png -compress jpeg -resize 1240x1753 -units PixelsPerInch -density 150x150 multipage.pdf


## References

* [convert images to pdf: How to make PDF Pages same size - Unix & Linux Stack Exchange](http://unix.stackexchange.com/a/20057)
* [ISO 216 (International standard paper sizes) - Wikipedia](https://en.wikipedia.org/wiki/ISO_216#A.2C_B.2C_C_comparison)