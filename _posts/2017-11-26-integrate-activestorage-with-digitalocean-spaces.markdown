---
layout: post
title:  "Integrate ActiveStorage with DigitalOcean Spaces"
date:   2017-11-26 21:19:30 +1000
---
I have been working on my side project comix for a while now. I decided to move it to Edge Rails because I wanted to use [ActiveStorage](https://github.com/rails/rails/tree/master/activestorage), a rails native library to upload files. Also, a major 5.2 release is closing in anyway.


DigitalOcean recently introduced [Spaces](https://www.digitalocean.com/products/object-storage/). Their private object storage is one to look out for. Mainly due to pricing - 5$ a month for 250 GB storage and 1TB transfer a month!

The best thing about the service is, it's API is fully compatible with S3. So, both can be used interchangeably.

**Moving the the app to Edge Rails:**

We need to first move our app from Rails 5.1.4 to Edge Rails. If you don't know what edge rails is, read more here.

So, point the rails branch to master in your ```Gemfile``` and run ```bundle install```. This will update a few gems like Arel and add new ones to your ```Gemfile.lock``` like ```ActiveStorage```.

{% highlight ruby %}
gem 'rails', git: 'https://github.com/rails/rails.git'
{% endhighlight %}

{% highlight shell %}
$ bundle install
......
Using arel 9.0.0 (was 8.0.0)
Using activerecord 5.2.0.alpha (was 5.1.4) from https://github.com/rails/rails.git (at master@c43bcc8)
Using activestorage 5.2.0.alpha from https://github.com/rails/rails.git (at master@c43bcc8)
.........
{% endhighlight %}

**Setup ActiveStorage:**

In order to install ActiveStorage, we first need to grab the migration. In order to do that, we need to run the install command.

{% highlight shell %}
$ rails active_storage:install
Copied migration 20171126083024_create_active_storage_tables.active_storage.rb from active_storage
$ rails db:migrate
== 20171126083024 CreateActiveStorageTables: migrating ========================
-- create_table(:active_storage_blobs)
   -> 0.0067s
-- create_table(:active_storage_attachments)
   -> 0.0101s
== 20171126083024 CreateActiveStorageTables: migrated (0.0170s) ===============
{% endhighlight %}

From ActiveStorage documentation, here is what the two table fields do:

>A key difference to how Active Storage works compared to other attachment solutions in Rails is through the use of built-in Blob and Attachment models (backed by Active Record). This means existing application models do not need to be modified with additional columns to associate with files. Active Storage uses polymorphic associations via the Attachment join model, which then connects to the actual Blob.

>Blob models store attachment metadata (filename, content-type, etc.), and their identifier key in the storage service. Blob models do not store the actual binary data. They are intended to be immutable in spirit. One file, one blob. You can associate the same blob with multiple application models as well. And if you want to do transformations of a given Blob, the idea is that you'll simply create a new one, rather than attempt to mutate the existing one (though of course you can delete the previous version later if you don't need it).

Assuming you have a digitalocean account, first create a space.

![Digitalocean menu](http://res.cloudinary.com/drg9hguhu/image/upload/v1511689099/Screen_Shot_2017-11-26_at_8.35.21_pm_lapq0a.png)

![Create space](http://res.cloudinary.com/drg9hguhu/image/upload/v1511689099/Screen_Shot_2017-11-26_at_8.34.56_pm_pnbhyo.png)

In order for ActiveStorage to connect to the service, we first need to create ```storage.yml``` inside the ```config``` folder of the app.