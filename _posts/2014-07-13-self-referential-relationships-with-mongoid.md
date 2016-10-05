---
layout: post
title:  "Self referential relationships with mongoid"
date:   2014-07-13 21:19:30 +1000
---
In several work scenarios we need a to create a parent-child relationship on a single model. So, it could be category and subcategory structure in an e-commerce site, or page and sub-page in a CMS. We will use category for our example. We will first generate our category model.

{% highlight shell %}
$ rails g model category title:string
      invoke  mongoid
      create    app/models/category.rb
      invoke    rspec
      create      spec/models/category_spec.rb
{% endhighlight %}

We need another model where we define the parent and child relationship of category model with itself. We will call this model category_relationship.

{% highlight shell %}
$ rails g model category_relationship
      invoke  mongoid
      create    app/models/category_relationship.rb
      invoke    rspec
      create      spec/models/category_relationship_spec.rb
{% endhighlight %}

In our category_relationship model, we will now define fields – category_id and child_id. child_id is the reference field for storing the id of the child category. We will define a belongs_to relationship for the parent on the category model and another belongs_to relationship for the child on the same model.

{% highlight ruby %}
app/models/category_relationship.rb

class CategoryRelationship
  include Mongoid::Document

  field :category_id, type: String
  field :child_id, type: String

  belongs_to :parent, class_name: "Category"
  belongs_to :child, class_name: "Category
end
{% endhighlight %}

In our category model, we will define relationships between parent and child. So, has_many and belongs_to relationships, both are made on the same model, and the reference happens via inverse_of.

{% highlight ruby %}
class Category
  include Mongoid::Document
  field :title, type: String

  has_many :child_category, class_name: 'Category', inverse_of: :parent_category, dependent: :destroy
  belongs_to :parent_category, class_name: 'Category', inverse_of: :child_category
end
{% endhighlight %}

Let’s create some category data.

{% highlight shell %}
$ rails c
Loading development environment (Rails 4.1.1)
2.1.1 :001 > category1 = Category.new(title: "Apparel")
 => #<Category _id: 53c2279f67617214f0000000, title: "Apparel", parent_category_id: nil>

2.1.1 :002 > category1.save
  MOPED: 127.0.0.1:27017 COMMAND      database=admin command={:ismaster=>1} runtime: 1.9274ms
  MOPED: 127.0.0.1:27017 INSERT       database=learning_development collection=categories documents=[{"_id"=>BSON::ObjectId('53c2279f67617214f0000000'), "title"=>"Apparel"}] flags=[]
                         COMMAND      database=learning_development command={:getlasterror=>1, :w=>1} runtime: 0.9229ms
 => true
 2.1.1 :003 > category2 = Category.new(title: "Men")
 => #<Category _id: 53c227d567617214f0010000, title: "Men", parent_category_id: nil>

2.1.1 :004 > category2.save
  MOPED: 127.0.0.1:27017 INSERT       database=learning_development collection=categories documents=[{"_id"=>BSON::ObjectId('53c227d567617214f0010000'), "title"=>"Men"}] flags=[]
                         COMMAND      database=learning_development command={:getlasterror=>1, :w=>1} runtime: 1.0399ms
 => true
{% endhighlight %}

We called our relationship parent_category and child_category. So we will use these terms to create and traverse the tree.

{% highlight shell %}
2.1.1 :005 > category2.parent_category = category1
 => #<Category _id: 53c2279f67617214f0000000, title: "Apparel", parent_category_id: nil>

2.1.1 :007 > category2.save
  MOPED: 127.0.0.1:27017 UPDATE       database=learning_development collection=categories selector={"_id"=>BSON::ObjectId('53c227d567617214f0010000')} update={"$set"=>{"parent_category_id"=>BSON::ObjectId('53c2279f67617214f0000000')}} flags=[]
                         COMMAND      database=learning_development command={:getlasterror=>1, :w=>1} runtime: 1.2537ms
 => true
{% endhighlight %}

Let’s see the result of the relationships we just created. First, we will check the parent.

{% highlight shell %}
2.1.1 :008 > category2
 => #<Category _id: 53c227d567617214f0010000, title: "Men", parent_category_id: BSON::ObjectId('53c2279f67617214f0000000')>

2.1.1 :009 > category2.parent_category
 => #<Category _id: 53c2279f67617214f0000000, title: "Apparel", parent_category_id: nil>
{% endhighlight %}

Then, we will check the child. Note that this returns an array, because of the has_many relationship.

{% highlight shell %}
2.1.1 :010 > category1.child_category
  MOPED: 127.0.0.1:27017 QUERY        database=learning_development collection=categories selector={"parent_category_id"=>BSON::ObjectId('53c2279f67617214f0000000')} flags=[] limit=0 skip=0 batch_size=nil fields=nil runtime: 1.1054ms

 => [#<Category _id: 53c227d567617214f0010000, title: "Men", parent_category_id: BSON::ObjectId('53c2279f67617214f0000000')>]
{% endhighlight %}
