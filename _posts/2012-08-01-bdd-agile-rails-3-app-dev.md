---
layout: post
title: "Agile (BDD) Rails 3 Application Development"
description: ""
category: development
tags: [agile, bdd, rails, best-practice]
---
{% include JB/setup %}

# The Skinny #

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
	$ git confit user.email <email>
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


# Reference #
[Manning: Rails 3 in Action](http://manning.com/katz)
