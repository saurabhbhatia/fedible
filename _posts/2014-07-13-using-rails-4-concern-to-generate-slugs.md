---
layout: post
title:  "Using rails 4 concern to generate slugs"
date:   2014-07-13 21:19:30 +1000
---
Though concerns have been around since Rails 3.2, they were extracted as a separate feature in Rails 4. At low-level, concerns are basically ruby modules that can be mixed with model or controller classes to make the defined methods available to that class in its context.

Slugs are an important feature in today’s apps where URLs play an important role for several reason. Search Engine Optimization, better user experience, easier bookmarking. There are gems available for creating slugs, like fiendly_id and mongoid-slug. However, a lot of times we need a custom solution. In rails it is easy enough to roll out our own.

We will first create a file for concern inside models.

{% highlight ruby %}
app/models/concerns/slug.rb

module Slug
  extend ActiveSupport::Concern

  def to_param
    self.title
  end
end
{% endhighlight %}

We will call this concern inside our model :

{% highlight ruby %}
app/models/product.rb

class Product
 include Mongoid::Document
 include Mongoid::Timestamps

 include Slug
 field :name, type: String
end
{% endhighlight %}

Now, this uses the default to_param method of Rails. Also, the url generated in this case for a title like, “this is a test”, will look like the following :

http://example.com/this-is-a-test

For SEO reasons, and for the reasons of localization, a lot of times URLs might have a UID.

e.g https://in.news.yahoo.com/arubas-leader-legislators-launch-hunger-strike-180842861.html

In order to generate uid, we will generate a random number and append it to the title. Also we will tie it to the callback in order to generate it before the record is created.

{% highlight ruby %}
module Slug
 extend ActiveSupport::Concern

 included do
   before_create :generate_uid
 end

 def to_param
   [self.title, self.uid].join("-")
 end

private

 def generate_uid
   self.uid = rand(36**8).to_s(36)
 end
end
{% endhighlight %}

A couple of things now remain. First, we need a field called title in order to create the slug. What if the model does not have such a field. In mongoid, we can assign a document field a different name. So a field called name, can also be alternatively called as slug.

{% highlight ruby %}
app/models/product.rb

field :name,as: slug,type: String
{% endhighlight %}

Likewise, we will have to change our slug method to suit this change :

{% highlight ruby %}
app/models/concerns/slug.rb

module Slug
 extend ActiveSupport::Concern

 included do
   before_create :generate_uid
 end

 def to_param
   [self.title, self.uid].join("-")
 end

private

 def generate_uid
   self.uid = rand(36**8).to_s(36)
 end
end
{% endhighlight %}

Lastly, we need to sanitize the string in our title. We will look for white spaces and place dashes instead of them, remove special characters from the string, remove entities.

{% highlight ruby %}
module Slug
 extend ActiveSupport::Concern

 included do
   before_create :generate_uid
 end

 def to_param
   [self.slug.gsub(/[ "'*@#$%^&amp;()+=;:.,?&gt;|\\&lt;~_!]/,'-').gsub(/-{2,}/,'-'), self.uid].join("-")
 end

private

 def generate_uid
   self.uid = rand(36**8).to_s(36)
 end
end
{% endhighlight %}

The resultant URL is something like the following:

http://example.com/this-is-a-test-79926128
