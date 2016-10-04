---
layout: post
title:  "Rails 4.1 Integration testing basics"
date:   2014-07-28 21:19:30 +1000
---
While working with Rails 4.1, I realized that the default unit test framework and minitest work very well together. So, I decided to look at several different scenarios in this context. We will start with creating a blank controller.

{% highlight shell %}
$ rails g controller welcome index
{% endhighlight %}

{% highlight ruby %}
app/controllers/welcome_controller.rb

class WelcomeController < ApplicationController
  def index
  end
end
{% endhighlight %}

In order to test the index response,the method generated is as follows.

{% highlight ruby %}
test/controllers/welcome_controller_test.rb

require 'test_helper'
require 'minitest/autorun'

class WelcomeControllerTest < ActionController::TestCase
  test "should get index" do
    get :index
    assert_response :success
  end
end
{% endhighlight %}

According to our specification, we need to make our welcome as route. We will first write a test for it. We need to create a test under the integration tests for the routes.

{% highlight ruby %}
test/integration/root_test.rb

require 'test_helper'
require 'minitest/autorun'

class RootTest < ActionController::TestCase
    test "should route to root" do
        assert_routing "/", {controller: "welcome", action: "index"}
    end
end
{% endhighlight %}

We will now try to run this test, but it will fail.

{% highlight shell %}
$ rake test
Run options: --seed 57718
# Running:
F..
Finished in 0.159120s, 18.8537 runs/s, 25.1383 assertions/s.

  1) Failure:
RootTest#test_0001_should route to root [/home/saurabh/mealr/test/integration/root_test.rb:6]:
No route matches "/"

3 runs, 3 assertions, 1 failures, 0 errors, 0 skips
{% endhighlight %}

We will edit our routes to make the test pass.

{% highlight ruby %}
config/routes.rb

Rails.application.routes.draw do
  root 'welcome#index'
end
{% endhighlight %}

Let’s run the tests and see how it goes.

{% highlight shell %}
$ rake test
Run options: --seed 42870
# Running:
...
Finished in 0.153732s, 19.5145 runs/s, 45.5338 assertions/s.

3 runs, 3 assertions, 0 failures, 0 errors, 0 skips
{% endhighlight %}

Alright, now we need to write a test for getting last 5 meals, sorted by available date.
In order to create this test, we need to create a fixture.

{% highlight ruby %}
test/fixtures/meals.yml

one:
  name: Pizza
  chef_id: 1
  meal_type_id: 1
  food_preference_id: 1
  availability: 2014-07-27 20:40:39

two:
name: Rissoto
  chef_id: 1
  meal_type_id: 1
  food_preference_id: 1
  availability: 2014-07-26 20:40:39

three:
  name: Pasta
  chef_id: 1
  meal_type_id: 1
  food_preference_id: 1
  availability: 2014-07-24 20:40:39


four:
  name: Burger
  chef_id: 1
  meal_type_id: 1
  food_preference_id: 1
  availability: 2014-07-24 20:40:39

five:
  name: Taco
  chef_id: 1
  meal_type_id: 1
  food_preference_id: 1
  availability: 2014-07-23 20:40:39

six:
  name: Burrito
  chef_id: 1
  meal_type_id: 1
  food_preference_id: 1
  availability: 2014-07-23 20:40:39
{% endhighlight %}

We have not created associations yet, so we can keep chef_id, meal_type_id and food_preference_id as ids. Also assert_equal will match the expected results and actual results. The test now looks like the following:

{% highlight ruby %}
test "should get last 5 meals sorted by available_date" do
  get :index
  assert_response :success
  expected_response = [meals(:one), meals(:two), meals(:three), meals(:four), meals(:five)]
  assert_equal expected_response, assigns(:meals)
  assert_equal 5, assigns(:meals).length
end
{% endhighlight %}

In order to pass this test, we need to write the code.

{% highlight ruby %}
app/controllers/welcome_controller.rb

class WelcomeController < ApplicationController
  def index
    @meals = Meal.order(availability: :desc).limit(5)
  end
end
{% endhighlight %}

Great! So we have tested the descending order and limit.
