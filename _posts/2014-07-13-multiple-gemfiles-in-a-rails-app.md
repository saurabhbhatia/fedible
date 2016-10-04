---
layout: post
title:  "Multiple Gemfiles in a rails app"
date:   2014-07-13 21:19:30 +1000
---
In a large rails app, with several extensions as rails engines, it is a good idea to abstract them into different file. This is to keep the gems required for the app separate from custom built rails engines. We need to create a file in the app folder, and load it inside the main Gemfile.

{% highlight ruby %}
..........
#default apps
eval(File.read(File.dirname(__FILE__) + '/default_apps.rb'))
#purchased apps
eval(File.read(File.dirname(__FILE__) + '/purchased_apps.rb'))
..........
{% endhighlight %}
