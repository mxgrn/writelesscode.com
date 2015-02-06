---
title:      "Building Rails-ExtJS widgets with Netzke, part 1"
created_at: 2008-12-13
kind: article
outdated: true
slug: "building_railsextjs_reusable_components_with_netzke_part_i"
tags: [tutorials, netzke]
---
<img class="frame-right" src="http://writelesscode.com/images/2008-12-13.jpg" alt="Building Rails/ExtJS Reusable Components With Netzke, Part 1"/>
This tutorial will show you how to create an AJAX-based Rails/ExtJS component using the netzke-core Rails plugin (the core part of the Netzke framework). It includes creating a demo Rails application from scratch. If you prefer to directly see the results (rather than follow the tutorial step-by-step), go to the last section below, and you'll be up-and-running in just 3 simple steps.

## Introduction
*This tutorial has been updated on **2010-01-10** to be compatible with the latest netzke-core*

We will create a Ext.tree.TreePanel-based widget from scratch and make it dynamically load the nodes from the server. A significant part of this tutorial is dealing with the basics of creating a Rails application, and it is there for those who are new to Rails. This tutorial requires Rails >= 2.3.0 and ExtJS >= 3.0.

## Preparing a Rails/ExtJS application, and installing Netzke
Let's start with creating a fresh rails application called "netzke-tutorial":

`rails netzke-tutorial && cd netzke-tutorial`

Download ExtJS from <http://extjs.com/products/extjs/download.php>, and link it into the application's public folder as `extjs` (make sure that, as the result, the folder "adapter" appears below the folder "public/extjs"), e.g.:

`ln -s ~/code/extjs/ext-3.1.0 public/extjs`

Install netzke-core as a Rails plugin (the latest and greatest version, as compared to the gem):

`./script/plugin install git://github.com/netzke/netzke-core.git`

Declare the Netzke routes somewhere in routes.rb:

~~~ruby
map.netzke
~~~

> This declares the routes that Netzke will be using for client-server communication (among the rest).

Generate a "welcome" controller with an "index" action:

`./script/generate controller welcome index`

Delete the Rails' index file (public/index.html).

Create a simple application layout (in 'app/views/layout/application.html.erb'):

~~~html
<head>
  <title>My first Netzke widget</title>
  <%%= netzke_init %>
</head>
<body>
  <%%= yield %>
</body>
~~~

> Note the `netzke_init` helper in the "header" section: it provides for all the necessary JS and CSS includes, both for ExtJS and Netzke. Enabling Netzke in your views is as simple as that.

Enough with the preparations, let's get to the fun part.

## Creating Netzke widget

Create the folder "lib/netzke", and inside it a file called "my\_tree.rb" - here goes the code for our widget. Define the widget class in it:

~~~ruby
class Netzke::MyTree < Netzke::Base
end
~~~

This is enough for the simplest widget (by default, it results in a JS class inheriting from `Ext.Panel`). Let's embed it into our index.html.erb:

~~~html
<h1>My tree panel</h1>
<div style='width: 300px;'>
  <%%= netzke :my_tree %>
</div>
~~~

As you see, embedding a Netzke widget into a view is done with the `netzke` helper. See the result in the browser. It's very basic, but it works, right?

You can specify the majority of options of the `Ext.Panel` component in the `:ext_config` hash, and this is where, I hope, it starts getting interesting. Try the following, for instance:

~~~ruby
<%%= netzke :my_tree, :ext_config => {
  :html => 'Just a simple panel for now',
  :title => 'Here comes the tree',
  :body_style => "padding: 5px;",
  :border => true
}%>
~~~

Ok, let's get rid of `:ext_config`, and make the JS part of our widget inherit from `Ext.tree.TreePanel` (instead of the default `Ext.Panel`). We do it by defining the `js_base_class` class method in our `MyTree` class:

~~~ruby
class Netzke::MyTree < Netzke::Base
  def self.js_base_class
    "Ext.tree.TreePanel"
  end
end
~~~

Also, `Ext.tree.TreePanel` requires a root to be defined, we do it in `js_extend_properties` class method:

~~~ruby
def self.js_extend_properties
  {
    :root => {:text => 'Root', :id => 'source'}
  }
end
~~~

> In `js_extend_properties` define all the properties that the JS class will have by default, including its public functions (including the overrides), such as `initComponent`, which we are going to define below.

Reload the page in the browser and see what's changed: now we have a tree!

## Add Rails model for tree data
In order to make the tree dynamically load its nodes from the server, we'll need a Rails model with `acts_as_tree` functionality mixed in (let's call it "Folder"):

`./script/generate model Folder name:string parent_id:integer`

Add `acts_as_tree` line into app/models/folder.rb, so that it looks like this:

~~~ruby
class Folder < ActiveRecord::Base
  acts_as_tree
end
~~~

Also, `acts_as_tree` comes as a Rails plugin, so we'll need to install it, too:

`./script/plugin install git://github.com/rails/acts_as_tree.git`

To seed some data into the tree, add the following into db/seeds.rb:

~~~ruby
# Initial tree data
root = Folder.create(:name => '/')
root.children.create(:name => 'One')
root.children.create(:name => 'Two').children.create(:name => 'Three')
~~~

Now, run the migrations and seed the data:

`rake db:migrate && rake db:seed`

## Server part of the widget
Now, how do we implement the communication between the browser and the server? Netzke makes it very easy. We need to declare the interface between the client and the server sides of the widget. Use the `api` class method of Netzke:

~~~ruby
class Netzke::MyTree < Netzke::Base
  api :get_children
  ...
end
~~~

It will automatically provide for the following. The client side of the widget will get the method `getChildren`, which will initiate the AJAX communication with the server side. Calling this method will result in a remote call to the widget's instance method `get_children`. This is explained in detail [here](http://wiki.github.com/netzke/netzke/client-server-communication-within-a-netzke-widget), and covered in the [third part of the tutorial](/blog/2009/09/24/building-rails-extjs-reusable-components-with-netzke-part-3/). However, in this particular case, we will not need to call the server side explicitly, but rather provide `Ext.tree.TreePanel` with a URL from which it will be able to fetch the data (see [Ext.tree.TreePanel documentation](http://www.extjs.com/deploy/dev/docs/?class=Ext.tree.TreePanel) for details). We'll do it by overriding the `initComponent` method (that any `Ext.Component` has):

~~~ruby
# New and overriden properties of the resulting JS class
def self.js_extend_properties
  {
    :root => {:text => 'Root', :id => 'source'},
    :init_component => <<-END_OF_JAVASCRIPT.l,
      function(){
        // dataUrl should be set to the API point which provides us with the nodes
        this.dataUrl = this.buildApiUrl("get_children");
        // Call the superclass' initComponent
        #{js_full_class_name}.superclass.initComponent.call(this);
      }
    END_OF_JAVASCRIPT
  }
end
~~~

> In order to retrieve the URL for a specific API point, we use Netzke's `buildApiUrl` method. This abstracts away the fact that each JS instance of a Netzke widget finds a way to connect to its proper Ruby instance. This is what provides us with high reusability (see the next section).

Now we only need to implement the server side of the interface:

~~~ruby
def get_children(params)
  klass = config[:model].constantize
  node = params[:node] == 'source' ? klass.find_by_parent_id(nil) : klass.find(params[:node].to_i)
  node.children.map{|n| {:text => n.name, :id => n.id}}
end
~~~

The method returns an array of tree nodes, which Netzke will translate into JSON format and pass to the client as a response to the AJAX request. You see that this method uses the configuration parameter "model", which we should provide to our widget so that it knows which Rails model should be used for the data. Add it to the `netzke` helper call in the view:

~~~html
<h1>My tree</h1>
<div style='width: 300px;'>
  <%%= netzke :my_tree, :model => "Folder" %>
</div>
~~~

That's it! Reload your browser page and see the result.

## Reusability of Netzke widgets
The best part of having a Netzke widget like this, is that it's very easily reusable. To start with, you may have multiple MyTree's in the same view (and Netzke is smart enough to not duplicate JS class definitions for each widget's instance). In order to do that, simply name them differently, and provide individual configuration for each one. Netzke will take care of each client-side instance of the widget talking to its proper server-side instance:

~~~html
<%%= netzke :my_first_tree, :class_name => "MyTree", :model => "Folder", :ext_config => {:title => "Folders"} %>
<%%= netzke :my_second_tree, :class_name => "MyTree", :model => "Categories", :ext_config => {:title => "Categories"} %>
~~~

> We didn't have to specify `class_name` option before, because, by convention, Netzke could infer it from the widget's name.

Further, you may use any Netzke widget as a part of a compound widget, as well as load it dynamically on the fly. For these techniques, see the next parts of this tutorial's series.

## For busy folks
Instead of following the above instructions step-by-step, you can simply do the following:

Get the tutorial source code:

`git clone git://github.com/netzke/netzke-tutorial.git`

Link your ExtJS library into public/extjs, e.g.:

`ln -s ~/code/extjs/ext-3.1.0 public/extjs`

Start the application:

`cd netzke-tutorial && ruby script/server`

That's it. Point your browser to <http://localhost:3000>

----
You can find the source-code for this tutorial on [GitHub](http://github.com/netzke/netzke-tutorial).

News on Netzke:

* Twitter: <http://twitter.com/nomadcoder>
* Blog: <http://writelesscode.com>
