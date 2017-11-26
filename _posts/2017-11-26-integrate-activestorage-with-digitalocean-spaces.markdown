---
layout: post
title:  "Integrate ActiveStorage with DigitalOcean Spaces"
date:   2017-11-26 21:19:30 +1000
---
I have been working on my side project [comix](https://comix.life) for a while now. I decided to move it to Edge Rails because I wanted to use [ActiveStorage](https://github.com/rails/rails/tree/master/activestorage), a rails native library to upload files. Also, a major 5.2 release is closing in anyway, so it would be a good idea to make the application compatible with it.


DigitalOcean recently introduced [Spaces](https://www.digitalocean.com/products/object-storage/). Their private object storage is one to look out for. Mainly due to pricing - 5$ a month for 250 GB storage and 1TB transfer a month!

The best thing about the service is, it's API is fully compatible with S3. So, both can be used interchangeably. A special shout out to [GoRails](https://gorails.com/episodes/digital-ocean-spaces-with-rails?autoplay=1), from where I got this idea.

**Moving the the app to Edge Rails:**

We need to first move our app from Rails 5.1.4 to Edge Rails. If you don't know what edge rails is, read more [here](http://edgeguides.rubyonrails.org/).

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

In order to install ActiveStorage, we first need to grab the migration by running the install command.

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

![Create space](https://res.cloudinary.com/drg9hguhu/image/upload/v1511689099/Screen_Shot_2017-11-26_at_8.34.56_pm_pnbhyo.png)

To connect to DigitalOcean API, we need to create API key and secret for spaces.

![Create API Keys](https://res.cloudinary.com/drg9hguhu/image/upload/v1511689784/Screen_Shot_2017-11-26_at_8.48.11_pm_xsb9mj.png)

For ActiveStorage to connect to the service, we first need to create ```storage.yml``` inside the ```config``` folder of the app. We need to setup our provider as amazon and S3 since, DigitalOcean API is fully compatible with S3.

{% highlight yml %}
amazon:
  service: S3
  access_key_id: <DigitalOcean Spaces API Key>
  secret_access_key: <DigitalOcean Spaces Secret>
  region: nyc3
  bucket: awesome-space
  endpoint: 'https://nyc3.digitaloceanspaces.com'
{% endhighlight %}

In your environments file,  ```config/environments/development.rb``` , add amazon as the service provider.

{% highlight ruby %}
  config.active_storage.service = :amazon
{% endhighlight %}

**Rails integration**

We'll start by making our model aware that there will be just one attachment associated with it.

{% highlight ruby %}
class Product < ApplicationRecord
  has_one_attached :product_image
end
{% endhighlight %}

Next, we need to go on and whitelist our file param in the controller, so that we can accept the object from the form.

{% highlight ruby %}
class ProductsController < ApplicationController
    private

    # Never trust parameters from the scary internet, only allow the white list through.
    def product_params
      params.require(:product).permit(:title, :description, :product_image)
    end
end
{% endhighlight %}

Finally, we will add the file upload field to our form.

{% highlight ruby %}
<%= form.file_field :product_image, class: "form-control" %>
{% endhighlight %}

We see the following when try to upload a file from our form.

{% highlight shell %}
Started POST "/products" for 127.0.0.1 at 2017-11-26 19:04:58 +1100
   (0.6ms)  SELECT "schema_migrations"."version" FROM "schema_migrations" ORDER BY "schema_migrations"."version" ASC
Processing by ProductsController#create as HTML
  Parameters: {"utf8"=>"✓", "authenticity_token"=>"", "product"=>{"title"=>"test 1", "description"=>"test test", "product_image"=>#<ActionDispatch::Http::UploadedFile:0x00007f8030e43b00 @tempfile=#<Tempfile:/var/folders/9g/x0rlkf_d1951361zltm54kjr0000gn/T/RackMultipart20171126-10783-3nsr2o.png>, @original_filename="Screen Shot 2017-11-13 at 1.51.45 pm.png", @content_type="image/png", @headers="Content-Disposition: form-data; name=\"product[product_image]\"; filename=\"Screen Shot 2017-11-13 at 1.51.45 pm.png\"\r\nContent-Type: image/png\r\n">, "store_id"=>"1"}, "commit"=>"Create Product"}
  S3 Storage (4420.2ms) Uploaded file to key: <key> (checksum: <checksum>)
   (0.2ms)  BEGIN
  ActiveStorage::Blob Create (7.4ms)  INSERT INTO "active_storage_blobs" ("key", "filename", "content_type", "byte_size", "checksum", "created_at") VALUES ($1, $2, $3, $4, $5, $6) RETURNING "id"  [["key", "<key>"], ["filename", "Screen Shot 2017-11-13 at 1.51.45 pm.png"], ["content_type", "image/png"], ["byte_size", 180926], ["checksum", "<checksum>"], ["created_at", "2017-11-26 08:05:03.141743"]]
   (0.7ms)  COMMIT
   (0.2ms)  BEGIN
  Store Load (0.4ms)  SELECT  "stores".* FROM "stores" WHERE "stores"."id" = $1 LIMIT $2  [["id", 1], ["LIMIT", 1]]
  Product Create (0.8ms)  INSERT INTO "products" ("title", "description", "store_id", "created_at", "updated_at") VALUES ($1, $2, $3, $4, $5) RETURNING "id"  [["title", "test 1"], ["description", "test test"], ["store_id", 1], ["created_at", "2017-11-26 08:05:03.213853"], ["updated_at", "2017-11-26 08:05:03.213853"]]
  Product Load (0.4ms)  SELECT  "products".* FROM "products" WHERE "products"."id" = $1 LIMIT $2  [["id", 2], ["LIMIT", 1]]
  ActiveStorage::Attachment Create (14.3ms)  INSERT INTO "active_storage_attachments" ("name", "record_type", "record_id", "blob_id", "created_at") VALUES ($1, $2, $3, $4, $5) RETURNING "id"  [["name", "product_image"], ["record_type", "Product"], ["record_id", 2], ["blob_id", 1], ["created_at", "2017-11-26 08:05:03.218654"]]
  Product Update All (0.5ms)  UPDATE "products" SET "updated_at" = '2017-11-26 08:05:03.234342' WHERE "products"."id" = $1  [["id", 2]]
   (5.0ms)  COMMIT
[ActiveJob] Enqueued ActiveStorage::AnalyzeJob (Job ID: 72802743-3cf8-4bbf-8d71-02445dbcf010) to Async(default) with arguments: #<GlobalID:0x00007f8030d85a60 @uri=#<URI::GID gid://comix/ActiveStorage::Blob/1>>
Redirected to http://localhost:3000/products/2
Completed 302 Found in 4632ms (ActiveRecord: 41.8ms)
{% endhighlight %}

Now, in order to display this in a page, we can access the entire object. using the following :

{% highlight ruby %}
<%= image_tag(@product.product_image) %>
{% endhighlight %}

That's it, now you can render your attachment, object in any page you want to. Logs show the following response from DigitalOcean.

{%highlight shell%}
app/views/products/show.html.erb:1:in `_app_views_products_show_html_erb__3935560012928142470_70094280869380'
Started GET "/products/2" for 127.0.0.1 at 2017-11-26 19:12:16 +1100
Processing by ProductsController#show as HTML
  Parameters: {"id"=>"2"}
  Product Load (0.2ms)  SELECT  "products".* FROM "products" WHERE "products"."id" = $1 LIMIT $2  [["id", 2], ["LIMIT", 1]]
  Rendering products/show.html.erb within layouts/application
  ActiveStorage::Attachment Load (0.4ms)  SELECT  "active_storage_attachments".* FROM "active_storage_attachments" WHERE "active_storage_attachments"."record_id" = $1 AND "active_storage_attachments"."record_type" = $2 AND "active_storage_attachments"."name" = $3 LIMIT $4  [["record_id", 2], ["record_type", "Product"], ["name", "product_image"], ["LIMIT", 1]]
  ActiveStorage::Blob Load (0.2ms)  SELECT  "active_storage_blobs".* FROM "active_storage_blobs" WHERE "active_storage_blobs"."id" = $1 LIMIT $2  [["id", 1], ["LIMIT", 1]]
  Rendered products/show.html.erb within layouts/application (7.1ms)
  Member Load (0.9ms)  SELECT  "members".* FROM "members" WHERE "members"."id" = $1 ORDER BY "members"."id" ASC LIMIT $2  [["id", 1], ["LIMIT", 1]]
  Rendered shared/_subheader.html.erb (0.3ms)
  Rendered shared/_header.html.erb (49.9ms)
  Rendered shared/_footer.html.erb (0.5ms)
Completed 200 OK in 478ms (Views: 468.9ms | ActiveRecord: 6.3ms)


Started GET "/rails/active_storage/blobs/eyJfcmFpbHMiOnsibWVzc2FnZSI6IkJBaHBCZz09IiwiZXhwIjpudWxsLCJwdXIiOiJibG9iX2lkIn19--3efb5fcb2b4e3f57f5caee793b796d902289c4b2/Screen%20Shot%202017-11-13%20at%201.51.45%20pm.png" for 127.0.0.1 at 2017-11-26 19:12:17 +1100
Processing by ActiveStorage::BlobsController#show as PNG
  Parameters: {"signed_id"=>"<signed_id>", "filename"=>"Screen Shot 2017-11-13 at 1.51.45 pm"}
  ActiveStorage::Blob Load (0.6ms)  SELECT  "active_storage_blobs".* FROM "active_storage_blobs" WHERE "active_storage_blobs"."id" = $1 LIMIT $2  [["id", 1], ["LIMIT", 1]]
  S3 Storage (4.0ms) Generated URL for file at key: <key> (asset-url)
Redirected to <asset-url>&response-content-type=image%2Fpng&X-Amz-Algorithm=AWS4-HMAC-SHA256&X-Amz-Credential=<cred>%2Fnyc3%2Fs3%2Faws4_request&X-Amz-Date=20171126T081217Z&X-Amz-Expires=300&X-Amz-SignedHeaders=host&X-Amz-Signature=<signature>
Completed 302 Found in 9ms (ActiveRecord: 0.6ms)
{%endhighlight%}