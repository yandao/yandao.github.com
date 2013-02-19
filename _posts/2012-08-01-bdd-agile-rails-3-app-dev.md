---
layout: post
title: "Agile (BDD) Rails 3 Application Development"
description: ""
category: development
tags: [agile, bdd, rails, best-practice]
---
{% include JB/setup %}

# Introduction #

Trying to consolidate the best practices for creating a Rails 3 application, in order to produce application that is as clean and maintainable as possible.

The main reference for this post is Chapter 3 of the book [Rails 3 in Action](http://manning.com/katz) by Manning.

For Behaviour-Driven Development (BDD) of Rails 3 application, we are going to use the [Cucumber](https://github.com/cucumber) testing framework.


# Create New Rails 3 Application #

	$ rails new <app_name>
	$ cd <app_name>

Alternatively, [Rails Composer](https://github.com/RailsApps/rails-composer) provides a template to create Rails apps by asking a series of questions (web server, database, test framework etc) and configure them from the beginning.
	
	$ rails new <app> -m https://raw.github.com/RailsApps/rails-composer/master/composer.rb
	
# Configure Git #

	$ git init
	$ git config user.name <name>
	$ git config user.email <email>
	$ git remote add origin <url>
	
	
# Configure RVM Environment #

Optional, if you have RVM installed and would like to create a separate gemset for each project, it might be good to add the **.rvmrc** script on the project root directory so that we won't be confused as to which gemset to use for a particular project.

	$ rvm --rvmrc --create <version>@<gemset>

The skeleton Rails 3 application already comes with pre-defined **.gitignore** file. Neat.


# Edit Gemfile to Include Gems Used for Testing #

Open the file Gemfile with your favourite text editor and include the following:

	group :test, :development do
	  gem 'rspec-rails', '~> 2.5'
	end

	group :test do
	  gem 'cucumber-rails'
	  gem 'capybara'
	  gem 'database_cleaner'
	end

Then call bundle install to install the gems that are defined in Gemfile with their dependencies:

	$ bundle install --binstubs


# Generate Skeleton for Cucumber and RSpec #

	$ rails g cucumber:install
	$ rails g rspec:install


# Configure Database #

Rails by default uses SQLite for databases. If we want to use another database, we need to modify config/database.yml.

Alternatively, we can specify which database to use in the very beginning when creating the application. For example, if we want to use MySQL:

	$ rails new <app> -d mysql2

At this point, we have more or less completed the bootstrapping of the Rails application. We can now start working on our project.


# Write Cucumber Feature #
	
In any application that makes use of persistent storage, it is almost imperative to implement the four basic CRUD functions.

Since we don't have anything yet, the first function to implement is the creation (C).
	
A Cucumber feature with scenario may look like this:

	Feature: Creating projects
	  In order to have projects to assign tickets to
	  As a user
	  I want to create them easily
	
	  Scenario: Creating a project
	    Given I am on the homepage
	    When I follow "New Project"
	    And I fill in "Name" with "TextMate 2"
	    And I press "Create Project"
	    Then I should see "Project has been created"

To run all features, run:
	
	$ rake cucumber:ok
	
If you want to follow everything in the book, it may be necessary to install older version of cucumber-rails, so edit it in Gemfile:

	group :test do
	  gem 'cucumber-rails', '= 1.0.6'
	...

If the database is not generated yet, run:

	$ rake db:migrate


# Re-route the Root Path #

The default root path in a Rails application points to the "Welcome aboard" page, which contains sensitive information about the application environment. So, it is advisable to re-route it to another path and delete the "Welcome aboard" page.

	$ git rm public/index.html

Using the scenario above, we can re-route the root path to point to the view for the resource called "**projects**". To do that, edit **config/routes.rb**, and add the following line before between the route definition:
	
	root :to => "projects#index"

Again, **projects** refer to the resource as defined in the scenario above.


# Create the MVC for the Resource #

Now, we need to define the MVC for the resource. Since the routing is handled by the Controller, we need to generate the controller:

	$ rails g controller <resource>

The command will generate a bunch of directories and files: the controller, the view, the helper, the RSpec test file etc. Note that the **model** is not generated here, so we are missing the M from the MVC.

Then we can proceed to define the actions (RESTful routing) of the controller for this resource by editing the file at app/controllers/&lt;resource&gt;\_controller.rb. In Rails, the controller typically has 7 actions:

* index
* show
* new
* create
* edit
* update
* destroy

NOTE: Run the command *rake cucumber:ok* repeatedly along the way to gain an understanding of how to build the MVC for a resource from scratch.

**TL;DR** Read Section 3.2 of "Rails 3 in Action"

In summary, the steps are:

1. Define resource routes in **config/routes.rb**.
2. Create the controller using **rails g controller &lt;resource&gt;**.
3. Define the action (index, show, new, create, edit, update, destroy) in the controller class.
4. Create the template in app/views/&lt;resource&gt;/&lt;action&gt;.html.&lt;template\_engine&gt;
5. Initialise the resource object in the **new** action in the controller class.
6. Create the model that talks directly with the persistent storage using **rails g model &lt;resource&gt; &lt;field&gt;:&lt;type&gt;**. A migration class for the resource will be generated, which allows us to modify the database schema directly by running **rake db:migrate**.


# Quick tips #

* Compare contents of two directories recursively. Useful if you want to build an app from scratch against a reference app, to find out which classes/files are missing/different.

	$ diff -qr <dir1> <dir2> | grep .git

* Check for empty directories (e.g. those automatically generated by rails commands but are unused) to reduce clutter:

	$ find . -type d -empty

# Reference #

* [Manning: Rails 3 in Action](http://manning.com/katz)
