---
layout: post
title: "Jekyll Bootstrap Setup"
description: ""
category: configuration
tags: [jekyll, twitter bootstrap]
---
{% include JB/setup %}

# Introduction #

Not really part of the main docs, but simply a checklist of things to look out for when changing the look and feel of this site.


# Items to modify in \_config.yml #

* Title, tagline, author's information (name, e-mail etc).
* To disable comments and analytics, set the providers to **false**.


# Modify Landing Page #

* The default landing page is **index.md**.
* It uses the **\_layouts/page.html** for layout, which in turn references the theme (**\_includes/themes**).


# Modify CSS #

* This is just a personal preference, so can be skipped.
* Jekyll seems to use &lt;h1&gt; tags for the main title only, and uses &lt;h2&gt; or smaller for the headings in the contents section.
* So if you use &lt;h1&gt; for the headings of the contents, it will appear rather blown up. To reduce the size, need to override it in the CSS.
* That CSS file is called **style.css** and is located in the **assets** directory.
* Since the contents are contained in a &lt;div&gt; called **rows**, just add the following in the CSS (values may be adjusted accordingly):

 		.row h1 {  
			font-size: 25px;   
			margin: 2px 0 2px;  
			text-align:left;  
		}  
		.row h2 {  
			font-size: 20px;  
			line-height: 36px;  
		}  
		.row h3 {  
			font-size: 16px;  
			line-height: 27px;	
		}  
		.row h4 {  
			font-size: 12px;  
			line-height: 18px;  
 		}

* Now the main headings should have smaller size compared to the page title despite both using the same &lt;h1&gt; tag.


# Tagline bug #

* There is a small bug for the tagline that appears next to the page title for posts. Basically it is defined in the beginning together with layout, title, tags etc, and is not displayed if it is not defined.
* However, the layout template for posts have it hard-coded as &lt;small&gt;Supporting tagline&lt;/small&gt;, therefore, you will always see these words.
* The template is located at **\_includes/themes/twitter**, just replace the first 3 lines from **page.html** to **post.html**.


# Remove the sample post #

		rm -rf \_posts/core-samples


# Adding Table of Contents #

* Documentation, especially those long ones, can really use an automatically generated Table of Contents.
* After several trial and errors, this is the one that works [Table of Contents jQuery Plugin](https://github.com/dcneiner/TableOfContents).
* Copy the js files (**jquery.tableofcontents.js** and **jquery.tableofcontents.min.js**) to the sub-directory **assets/js**.
* Modify the template files **default.html** and **post.html**. **default.html** contains the &lt;head&gt; section where we include the Javascript, whereas **post.html** defines the layout for the contents.
* Copy the following lines somewhere between the &lt;head&gt;...&lt;/head&gt; in **default.html**:
  
		<script src="http://ajax.googleapis.com/ajax/libs/jquery/1.7.2/jquery.min.js" type="text/javascript"></script>
		<script src="/assets/js/jquery.tableofcontents.min.js" type="text/javascript"></script>

		<script type="text/javascript" charset="utf-8">
			$(document).ready(function(){ $("#toc").tableOfContents(
				null,
				{
					depth: 2,
				}
			); });
		</script>

* Add this line in **post.html** where you want the ToC to appear:

		<ul id="toc"></ul>

# Changing the Theme #

* If for some reason, you need to change the theme, simply create a sub-directory under **\_includes/themes** with the necessary files.