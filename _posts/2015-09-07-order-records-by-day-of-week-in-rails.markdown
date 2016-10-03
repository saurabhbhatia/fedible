---
layout: post
title:  "Order records by day of week in rails"
date:   2015-09-07 21:19:30 +1000
---
I recently had a small challenge to order the records of a model by “day of week” .  Records with day of week as Monday should appear on the top and stack multiple records of the same day.

**Setting up the models:**

In order to achieve this, I took a very simple approach. I use, Activerecord::Enum module in rails to build this. I first added a column called day_of_week

{% highlight shell %}
$ rails g migration add_day_of_week_to_class_schedules day_of_week:integer
{% endhighlight %}

In the class_schedule model, we would need to define the enum with the days of the week :

{% highlight ruby %}
class ClassSchedule < ActiveRecord::Base
 enum day_of_week: [:monday, :tuesday, :wednesday, :thursday, :friday, :saturday, :sunday]
end
{% endhighlight %}

The order in enum definition is extremely important. So in our database, data in day_of_week column would look something like the following :

Monday = 0 .. Sunday = 6

In order to add this as a selectbox in our activeadmin form :

{% highlight ruby %}
ActiveAdmin.register ClassSchedule do
  form html: { multipart: true } do |f|
    f.inputs do
     .... other inputs ...
      f.input :day_of_week, as: :select, collection: ClassSchedule.day_of_weeks.keys, label: "Day"
    end
   f.actions
  end
  permit_params  :day_of_week
end
{% endhighlight %}
