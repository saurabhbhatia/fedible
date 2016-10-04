---
layout: post
title:  "Custom Login with Rails Part-1"
date:   2014-07-15 21:19:30 +1000
---
We will start with creating a model for the user. We need to create a field called password_digest for storing the encrypted password.

{% highlight shell %}
$ rails g model user user_name:string password_digest:string
      invoke  mongoid
      create    app/models/user.rb
      invoke    test_unit
      create      test/models/user_test.rb
      create      test/fixtures/users.yml
{% endhighlight %}

We need to add ‘bcrypt’ gem to the Gemfile as it is used by rails to encrypt the password internally.

{% highlight ruby %}
Gemfile

gem 'bcrypt'
{% endhighlight %}

ActiveModel::SecurePassword is a module required for generating and validating passwords in rails. In order to enable this module on a particular model, we need to include it. Then access these methods using has_secure_password.

{% highlight ruby %}
app/models/user.rb

class User
  include Mongoid::Document
  include Mongoid::Timestamps
  include ActiveModel::SecurePassword

  field :user_name, type: String
  field :password_digest, type: String

  has_secure_password
end
{% endhighlight %}

We have already added the ability to create users, password and password confirmation to our system. Let’s quickly check what we have done.

{% highlight shell %}
$ rails c
Loading development environment (Rails 4.1.1)
2.1.1 :001 > u = User.new
 => #
2.1.1 :002 > u.user_name = "saurabh"
 => "saurabh"
2.1.1 :003 > u.password = "123456"
 => "123456"
2.1.1 :004 > u.password_confirmation = "123456"
 => "123456"
2.1.1 :005 > u.save
  MOPED: 127.0.0.1:27017 COMMAND      database=admin command={:ismaster=>1} runtime: 2.0463ms
  MOPED: 127.0.0.1:27017 INSERT       database=learning_development collection=users documents=[{"_id"=>BSON::ObjectId('53c24217676172180d000000'), "user_name"=>"saurabh", "password_digest"=>"$2a$10$AvOo2g.RD4Sb31xHzspcJe34uzz6tY9roaZHQoYU4nwWZN8GyCe1C", "updated_at"=>2014-07-13 08:24:23 UTC, "created_at"=>2014-07-13 08:24:23 UTC}] flags=[] COMMAND database=learning_development command={:getlasterror=>1, :w=>1} runtime: 5.5310ms
 => true
{% endhighlight %}

In order to handle the creation of users, we will create a controller.

{% highlight shell %}
$ rails g controller users new
      create  app/controllers/users_controller.rb
       route  get 'users/new'
      invoke  erb
      create    app/views/users
      create    app/views/users/new.html.erb
      invoke  test_unit
      create    test/controllers/users_controller_test.rb
      invoke  helper
      create    app/helpers/users_helper.rb
      invoke    test_unit
      create      test/helpers/users_helper_test.rb
      invoke  assets
      invoke    coffee
      create      app/assets/javascripts/users.js.coffee
      invoke    scss
      create      app/assets/stylesheets/users.css.scss
{% endhighlight %}

We also need to setup the routes for users and create a root page. We will modify the users route to make it restful.

{% highlight ruby %}
config/routes.rb

Rails.application.routes.draw do
  resources :users
  root 'home#index'
end
{% endhighlight %}

Let’s modify the controller and create a new and create method. Make sure you whitelist a limited set of params not permit all.

{% highlight ruby %}
app/controllers/users_controller.rb

class UsersController < ApplicationController
  def new
    @user = User.new
  end

  def create
    @user = User.new(user_params)
      if @user.save
    redirect_to root_path
    flash["notice"] = "Signed Up Successfully"
      else
    render "new"
        flash["error"] = "Problems with your Signup"
      end
  end

  def user_params
     params.require(:user).permit(:password, :password_confirmation, :user_name, :email)
  end
end
{% endhighlight %}

In order to create signup for the user, we need to add a form for it.

{% highlight ruby %}
app/views/users/new.html.erb

<% if flash["notice"].present? -%>
   <%= flash[:notice]%>
<% end -%>

<%= form_for @user do |f|%>

   <label>User Name</label>
   <%= f.text_field :user_name %>

   <label>Email</label>
   <%= f.email_field :email %>

   <label>Password</label>
   <%= f.password_field :password %>

   <label>Password Confirmation</label>
   <%= f.password_field :password_confirmation %>

   <%= f.submit %>
<% end %>
{% endhighlight %}

We will create session and session objects in the next part.
Update: You can read the Part 2 of the tutorial here.
