---
layout: post
title:  "pass environment variables inside a rake task"
date:   2014-07-13 21:19:30 +1000
---
If you want to run you rake task without passing the environment variable in the command line argument.

Use Case: The task runs only in production and it needs to be automated.However, we need to pass a command line variable for the task to run.

Following is the task:

{% highlight ruby %}
namespace :my_task do
   desc "This is a task that runs only in production"
   task :awesome do
      if Rails.env.production?

      end
   end
end
{% endhighlight %}

The command used to run the task is as follows :

{% highlight shell %}
$ bundle exec rake my_task:awesome RAILS_ENV="production"
{% endhighlight %}

We can achieve this by writing another rake task that passes the environment variable to the respective task(s). In case there are a bunch of other tasks you can define them all under the same task.

{% highlight ruby %}
namespace :my_main_task do
  desc "Use this to fire other tasks"
  task : do
     Rails.env = "Production"
     Rake::Task["my_task:awesome"].invoke
  end
end
{% endhighlight %}

Or you can make a direct command call:

{% highlight ruby %}
namespace :orbit do
  desc "display the current environment of rake"
  task :asset do
     Rails.env = "Production"
     system("rake orbit_task:awesome RAILS_ENV=production")
  end
end
{% endhighlight %}

The issue with the second approach is that it will spawn a lot of rake tasks, so the first approach seems better.
