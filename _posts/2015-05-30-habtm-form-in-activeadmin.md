---
layout: post
title:  "HABTM form in Activeadmin"
date:   2015-05-30 21:19:30 +1000
---
I am building a cms & ecommerce project with an aim to keep the development fast and a basic feature set. ActiveAdmin is a good choice when it comes to a rails admin backend. It is configurable as well as customizable. Let’s see how to build an has_and_belongs_to_many form with it.

**Scenario:**
In a common store scenario, products can belong to many categories, and a category has many products.

**Setting up the models:**
We will start by adding has_and_belongs_to_many relationship in both product and category models.

{% highlight ruby %}
app/models/product.rb

class Product < ActiveRecord::Base
  has_and_belongs_to_many :category, join_table: :products_categories
end

app/models/category.rb

class Product < ActiveRecord::Base
  has_and_belongs_to_many :category, join_table: :products_categories
end
{% endhighlight %}

Join table is a reference table that holds product and categories for the lookup. Once, these have been setup, we need create the join table and model.

{% highlight shell %}
example-app$ rails g model products_category product_id:integer category_id:integer
example-app$ rake db:migrate
{% endhighlight %}

In the products_category model, we would need to setup the relationships as well :

{% highlight ruby %}
class ProductsCategory < ActiveRecord::Base
  belongs_to :product
  belongs_to :category
end
{% endhighlight %}

Our migration looks like the following :

{% highlight ruby %}
class CreateProductsCategories < ActiveRecord::Migration
  def change
    create_table :products_categories do |t|
      t.integer :product_id
      t.integer :category_id

      t.timestamps null: false
    end
  end
end
{% endhighlight %}

Making it work includes 2 parts :

- The form should be able to accept params and store it in products_categories table.
- A visual input to select multiple categories.

In order to fix the first bit, we will add category_ids as param. This will ensure, the array of category ids are picked up from the form and passed to the right table.

{% highlight ruby %}
ActiveAdmin.register Product do
  permit_params :title , category_ids: []
end
{% endhighlight %}

Lastly add a way to accept these from a form :
{% highlight ruby %}
ActiveAdmin.register Product do
  form html: { multipart: true } do |f|
    f.inputs do
      f.input :title, label: "Title"
      f.input :category_ids, as: :check_boxes, collection: Category.all
    end
    f.actions
  end

  # params whitelisting
  permit_params :title, category_ids: []
end
{% endhighlight %}
