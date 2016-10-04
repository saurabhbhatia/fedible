---
layout: post
title:  "Custom Login with Rails Part-2"
date:   2014-07-20 21:19:30 +1000
---
In the previous part we saw how to build the signup page. In this part we will create sessions and persist our user object in the session. To start this, we will create a controller to handle the sessions.

{% highlight shell %}
$ rails g controller sessions new
      create  app/controllers/sessions_controller.rb
       route  get 'sessions/new'
      invoke  erb
      create    app/views/sessions
      create    app/views/sessions/new.html.erb
      invoke  test_unit
      create    test/controllers/sessions_controller_test.rb
      invoke  helper
      create    app/helpers/sessions_helper.rb
      invoke    test_unit
      create      test/helpers/sessions_helper_test.rb
      invoke  assets
      invoke    coffee
      create      app/assets/javascripts/sessions.js.coffee
      invoke    scss
      create      app/assets/stylesheets/sessions.css.scss
{% endhighlight %}

Modify the routes to create resources for the sessions controller.

{% highlight ruby %}
config/routes.rb

Rails.application.routes.draw do
  resources :users
  resources :sessions
  root 'home#index'
end
{% endhighlight %}

In sessions controller, we will modify our actions to check the user by user_name. In case you want, you can change it to email or both. Rails does some more magic behind the scenes using the user.authenticate(params[:password]). has_secure_password generates the salt and allows us to use .authenticate method which matches the salted password with the params passed. Also, we will persist the user_id in the session. In order to create a logout, we will simply set the session’s user_id to nil.

{% highlight ruby %}
app/controllers/sessions_controller.rb

class SessionsController < ApplicationController
  def new
  end

  def create
    user = User.find_by(user_name: params[:user_name]) rescue nil
    if (user && user.authenticate(params[:password]))
      session[:user_id] = user.id.to_s
      redirect_to dashboards_path
    else
      flash.now.alert = "Invalid Username or Password"
      render "new"
    end
  end

  def destroy
    session[:user_id] = nil
    redirect_to root_url, :notice => "Logged out!"
  end
end
{% endhighlight %}

Our login form will look like the following and will be accessible at sessions/new.

{% highlight ruby %}
app/views/sessions/new.html.erb

<% flash.each do |key, msg| %>
 <%= content_tag :p, msg, :class => [key] %>
<% end %>

<%= form_tag sessions_path do %>
 <label>User Name</label>
 <%= text_field_tag :user_name, params[:user_name] %>

 <label>Password</label>
 <%= password_field_tag :password, nil %>

 <%= submit_tag "Login"%>
<% end %>
{% endhighlight %}

We will also create a separate route for logout.

{% highlight ruby %}
config/routes.rb

Rails.application.routes.draw do

  resources :users
  resources :sessions

  get "logout", to: "sessions#destroy", as: "logout"

  resources :dashboards
  root 'home#index'
end
{% endhighlight %}

We will create a blank controller where we can redirect the page after the session is created.

{% highlight shell %}
$ rails g controller dashboards index
      create  app/controllers/dashboards_controller.rb
       route  get 'dashboards/index'
      invoke  erb
      create    app/views/dashboards
      create    app/views/dashboards/index.html.erb
      invoke  test_unit
      create    test/controllers/dashboards_controller_test.rb
      invoke  helper
      create    app/helpers/dashboards_helper.rb
      invoke    test_unit
      create      test/helpers/dashboards_helper_test.rb
      invoke  assets
      invoke    coffee
      create      app/assets/javascripts/dashboards.js.coffee
      invoke    scss
      create      app/assets/stylesheets/dashboards.css.scss
{% endhighlight %}

Now, to access the persisted user as an object in the session we will create a current_user method. Through this we can access the user’s details when the he or she is in the session.

{% highlight ruby %}
app/controllers/application_controller.rb

class ApplicationController < ActionController::Base
  # Prevent CSRF attacks by raising an exception.
  # For APIs, you may want to use :null_session instead.
  protect_from_forgery with: :exception
  helper_method :current_user

  private

  def current_user
    @current_user ||= User.find(session[:user_id]) if session[:user_id]
  end
end
{% endhighlight %}

We also need a filter method to protect our methods that we need to keep only for the logged in Users. In order to do that, we will create a method called authenticate_user. This method will check the presence of user_id and accordingly redirect to the respective page.

{% highlight ruby %}
app/controllers/application_controller.rb

class ApplicationController < ActionController::Base
  # Prevent CSRF attacks by raising an exception.
  # For APIs, you may want to use :null_session instead.
  protect_from_forgery with: :exception
  helper_method :current_user

  private

  def current_user
    @current_user ||= User.find(session[:user_id]) if session[:user_id]
  end

  protected

  def authenticate_user
    if session[:user_id]
      # set current user object to @current_user object variable
      @current_user = User.find(session[:user_id]) rescue nil
      return true
    else
      redirect_to new_session_path
      return false
    end
  end
end
{% endhighlight %}

We can now put our dashboard behind the login.

{% highlight ruby %}
app/controllers/dashboards_controller.rb

class DashboardsController < ApplicationController
  before_action :authenticate_user

  def index
  end
end
{% endhighlight %}

Also, we can use current_user object to show conditional login and logout links and also show the details of the session user.

{% highlight ruby %}
app/views/layouts/application.html.erb

<!DOCTYPE html>
<html>
<head>
  <title>Login App</title>
  <%= stylesheet_link_tag    'application', media: 'all', 'data-turbolinks-track' => true %>
  <%= javascript_include_tag 'application', 'data-turbolinks-track' => true %>
  <%= csrf_meta_tags %>
</head>
<body>

<% if current_user.present? %>
  <%= current_user.user_name %> <%= link_to "Logout", logout_path%>
<%else%>
  <%= link_to "Login", new_session_path %>
<% end %>

<%= yield %>

</body>
</html>
{% endhighlight %}

So, we have successfully created sessions, made method to protect actions and also persist the user in the session.
