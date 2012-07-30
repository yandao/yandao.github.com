---
layout: post
title: "Ruby on Rails 3 REST API Example"
description: ""
category: tutorial
tags: [rails, rails3, rest]
---
{% include JB/setup %}

# The Skinny #

This post shows an example on how to create a simple Rails application that access resources from a MySQL database.

Read [this](/configuration/2012/07/30/mysql-on-os-x) for instructions to set up MySQL on OS X.

# Install MySQL gem #

Assuming MySQL is installed using Homebrew:

	$ env ARCHFLAGS="-arch x86_64" gem install mysql2 -- --with-mysql-config=/usr/local/Cellar/mysql/<version>/bin/mysql_config 

# Create Rails App with MySQL Database Access #

	$ rails new <app> -d mysql

# Configure Rails for Database Access #

At this point, the database must already exist with all privileges granted to a (preferably) non-root user. The next step is to enter the information in the configuration file:

	$ vim config/database.yml

Enter the database, user and password (plus other parameters as necessary):

	# MySQL
	  # gem install mysql2
	  development:
	    adapter: mysql2
	    host: localhost
	    database: <database>
	    username: <user>
	    password: <password>
	    pool: 5
	    timeout: 5000

# Create Table From Rails #

	$ rails g scaffold <table> <field1>:<type1> <field2>:<type2> ...
	$ rake db:migrate
			
# Test App UI #

	$ rails s
	$ open http://localhost:3000/<table>

In the web browser, you should see a default page that shows the existing records in the specified table, as well as a button to add new records. Test to see if it works.

(Optional) Login to MySQL console to verify that the records are correct.

# DRY-er Controller Class #

The default controller class created from scaffold contains many repeated parts, which should be re-factored to adhere to the DRY (Don't Repeat Yourself) principle. The result is a cleaner and much shorter piece of code too. The class is contained in the file **app/controllers/&lt;table&gt;\_controller.rb**, so open it with your favourite text editor and edit away.
	
The following example shows a DRY-er version of controller class for accessing the database table 'users'.
	
First step is to create helper methods representing the database table:

	private
  
	# Make these helpers available to the view too
	helper_method :user, :users
  
	def user
	  # If the action is new or create...
	  if @user = params[:action] =~ /new|create/
		User.new(params[:user])
	  else
		User.find(params[:id])
	  end
	end

	def users
	  @users = User.all
	end
	
Then the five API calls: index, show, create, update and destroy can be cleaned up to contain 1-2 lines of code only:

	respond_to :html, :json, :xml
    
	def index
	  respond_with users
	end

	def show
	  respond_with user
	end

	def create
	  user.save
	  respond_with user
	end

	def update
	  user.update_attributes(params[:user])
	  respond_with user
	end

	def destroy
	  user.destroy
	  respond_with user
	end

The complete listing:

	class UsersController < ApplicationController

	  respond_to :html, :json

	  def index
	    respond_with users
	  end

	  def show
	    respond_with user
	  end

	  def create
	    user.save
	    respond_with user
	  end

	  def update
	    user.update_attributes(params[:user])
	    respond_with user
	  end

	  def destroy
	    user.destroy
	    respond_with user
	  end

	  private

	  # Make these helpers available to the view too
	  helper_method :user, :users

	  def user
	    # If the action is new or create...
	    if @user = params[:action] =~ /new|create/
	      User.new(params[:user])
	    else
	      User.find(params[:id])
	    end
	  end

	  def users
	    @users = User.all
	  end

	end


# Reference #

* [How to write a Ruby and Rails 3 REST API](http://squarism.com/2011/04/01/how-to-write-a-ruby-rails-3-rest-api/)