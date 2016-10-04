---
layout: post
title:  "Setting up guard with minitest"
date:   2015-07-28 21:19:30 +1000
---
Guard is a tool to autorun tests as and when they are created. It basically runs a background process that continuously checks for the tests. As soon as they are created, it runs and provides the results. I feel it is an essential thing to increase the pace of work with the tests.

We will start by adding guard and guard-minitest to the Gemfile. Make sure you also add “ruby-prof” to testing group in order to install the dependencies correctly.

{% highlight ruby %}
group :development do
  gem 'guard'
  gem 'guard-minitest'
end

group :test do
  gem "minitest"
  gem 'minitest-spec-rails'
  gem 'ruby-prof'
end
{% endhighlight %}

Once, the installation is done, we need to generate a configuration file called Guardfile, for minitest.

{% highlight shell %}
$ guard init minitest
{% endhighlight %}

We can also uncomment the Rails 4 test watch rules. So, guard will now watch minitest unit and Rails 4 unit tests. In case you want to autorun minitest specs, you can also uncomment that part.

{% highlight ruby %}
  guard :minitest do
    watch(%r{^test/(.*)\/?test_(.*)\.rb$})
    watch(%r{^lib/(.*/)?([^/]+)\.rb$}) { |m| "test/#{m[1]}test_#{m[2]}.rb" }
    watch(%r{^test/test_helper\.rb$}) { 'test' }
    watch(%r{^app/(.+)\.rb$}) { |m| "test/#{m[1]}_test.rb" }
    watch(%r{^app/controllers/application_controller\.rb$}) { 'test/controllers' }
    watch(%r{^app/controllers/(.+)_controller\.rb$}) { |m| "test/integration/#{m[1]}_test.rb" }
    watch(%r{^app/views/(.+)_mailer/.+}) { |m| "test/mailers/#{m[1]}_mailer_test.rb" }
    watch(%r{^lib/(.+)\.rb$}) { |m| "test/lib/#{m[1]}_test.rb" }
    watch(%r{^test/.+_test\.rb$})
    watch(%r{^test/test_helper\.rb$}) { 'test' }
  end
{% endhighlight %}

We will try and run our guard now. Make sure you run the command using bundle exec guard else guard will throw a warning. This is to inform guard that we are using bundler to manage our gems.
