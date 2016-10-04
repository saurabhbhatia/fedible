---
layout: post
title:  "Rails engines with Mongoid"
date:   2014-07-13 21:19:30 +1000
---
In order to create a rails engine that loads mongoid by default instead of activerecord, we will start with creating a full rails engine with an option -O. This is to skip the inclusion of any database settings or active record.

{% highlight shell %}
$ rails plugin new new_engine --full -O
      create  
      create  README.rdoc
      create  Rakefile
      create  new_engine.gemspec
      create  MIT-LICENSE
      create  .gitignore
      create  Gemfile
      create  app/models
      create  app/models/.keep
      create  app/controllers
      create  app/controllers/.keep
      create  app/views
      create  app/views/.keep
      create  app/helpers
      create  app/helpers/.keep
      create  app/mailers
      create  app/mailers/.keep
      create  app/assets/images/new_engine
      create  app/assets/images/new_engine/.keep
      create  config/routes.rb
      create  lib/new_engine.rb
      create  lib/tasks/new_engine_tasks.rake
      create  lib/new_engine/version.rb
      create  lib/new_engine/engine.rb
      create  app/assets/stylesheets/new_engine
      create  app/assets/stylesheets/new_engine/.keep
      create  app/assets/javascripts/new_engine
      create  app/assets/javascripts/new_engine/.keep
      create  bin
      create  bin/rails
      create  test/test_helper.rb
      create  test/new_engine_test.rb
      append  Rakefile
      create  test/integration/navigation_test.rb
  vendor_app  test/dummy
         run  bundle install
Fetching gem metadata from https://rubygems.org/...........
Fetching additional metadata from https://rubygems.org/..
Resolving dependencies...
................
{% endhighlight %}

Despite this, as rails loads all the railties, it even loads activerecord with it.Hence, we need to customize our list of railties in rails/bin file.

{% highlight ruby %}
rails/bin

#!/usr/bin/env ruby
# This command will automatically be run when you run "rails" with Rails 4 gems installed from the root of your application.

ENGINE_ROOT = File.expand_path('../..', __FILE__)
ENGINE_PATH = File.expand_path('../../lib/new_engine/engine', __FILE__)

# Set up gems listed in the Gemfile.
ENV['BUNDLE_GEMFILE'] ||= File.expand_path('../../Gemfile', __FILE__)
require 'bundler/setup' if File.exist?(ENV['BUNDLE_GEMFILE'])

require "action_controller/railtie"
require "action_mailer/railtie"
require "sprockets/railtie"
require "rails/test_unit/railtie"
require "mongoid"
require 'rails/engine/commands'
{% endhighlight %}

We will now try to create a model inside the engine.

{% highlight shell %}
$ rails g model product title:string
bin/rails:15:in `require': cannot load such file -- mongoid (LoadError)
    from bin/rails:15:in `<main>'
{% endhighlight %}

In order to load mongoid, we will add it as a dependency to the gem. Make sure you use the same version of mongoid inside your application and rails engine.

{% highlight ruby %}
new_engine.gemspec

$:.push File.expand_path("../lib", __FILE__)

# Maintain your gem's version:
require "new_engine/version"

# Describe your gem and declare its dependencies:
Gem::Specification.new do |s|
  s.name        = "new_engine"
  s.version     = NewEngine::VERSION
  s.authors     = ["TODO: Your name"]
  s.email       = ["TODO: Your email"]
  s.homepage    = "TODO"
  s.summary     = "TODO: Summary of NewEngine."
  s.description = "TODO: Description of NewEngine."
  s.license     = "MIT"

  s.files = Dir["{app,config,db,lib}/**/*", "MIT-LICENSE", "Rakefile", "README.rdoc"]
  s.test_files = Dir["test/**/*"]

  s.add_dependency "rails", "~> 4.1.1"
  s.add_dependency 'mongoid', '4.0.0'
end
{% endhighlight %}

At this point we will run bundle install again and try to create a model.

{% highlight shell %}
$ rails g model product title:string
      invoke  mongoid
      create    app/models/product.rb
      invoke    test_unit
      create      test/models/product_test.rb
      create      test/fixtures/products.yml
{% endhighlight %}

Voila! You can now develop your engine using mongoid.
