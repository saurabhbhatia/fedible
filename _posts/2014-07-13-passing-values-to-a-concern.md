---
layout: post
title:  "Passing values to a concern"
date:   2014-07-13 21:19:30 +1000
---
In order to pass values to a controller concern, we first need to create a variable and bind it to a callback. We will define a variable called app_title and pass it to a method in the concern called fetch_variables. We will use class initialize method to define the variable value.

{% highlight ruby %}
app/controllers/admin/custom_app_controller.rb

class Admin::CustomAppController < AdminController
  include Authorize
  before_action ->(app_title = @app_title) { fetch_variables app_title }

  def initialize
    super
    @app_title = "custom_app"
  end
end
{% endhighlight %}

Now we will define a method in the concern to find and set variable on the callback.

{% highlight ruby %}
app/controllers/concerns/authorize.rb

module Authorize
  extend ActiveSupport::Concern

  private

  def fetch_variables(app_title)
    @app = App.find_by(key: app_title)
    @app_categories = @app.categories
  end
end
{% endhighlight %}
