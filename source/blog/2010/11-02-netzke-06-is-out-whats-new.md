--- 
title:      "Netzke 0.6 is out: what's new"
created_at: 2010-11-02
kind: article
tags: [netzke]
excerpt: This post highlights the most important changes that found their way into version 0.6.
--- 
<img align="right" class="frame-right" width="320" height="197" src="http://writelesscode.com/images/2010-11-01-1.jpg" alt="Netzke progress on GitHub"/>About 7 weeks ago I joined the development team of a Berlin based company called [Familien Service](http://www.familienservice.de/) to help them start with a long-term project where they would use Netzke as the basis for a fairly complex CRM-application. Thanks to constant feedback from their really talented guys, as well as random help from other contributors on GitHub, Netzke has made some significant progress, resulting in that a week ago, after [presenting Ext JS and Netzke":http://www.slideshare.net/netzke/rails-ext-js-and-netzke at "Ruby and Rails european conference](http://rubyandrails.eu/articles/introduction-to-netzke--2) in Amsterdam, I released the version 0.6.0 of both netzke-core and netzke-basepack. In this post I'd like to highlight the most important changes that found their way into the release.

## Rails 3 support and test coverage

The long awaited move to Rails 3 has happened! Many parts of the code, especially in netzke-core, have been profoundly refactored and restructured, and this time I took the test-driven approach, using Capybara and Cucumber to unit-test the components, along with RSpec for testing non-GUI functionality. I can't express enough how much confidence a good test coverage gives you as a developer, _especially_ if you strive to constantly improve the code by refactoring it, and _even more_ as the project starts receiving contributions. As the result, the current code makes it much easier for other developers to understand and improve the framework, and it shows by an increased amount of pull-requests I'm receiving on GitHub. Yet, there's a lot more work needed - you could still see some old code hanging around, and there are places here and there that cry out for refactoring.

## Basepack components get name-spaced

All the components that were provided by netzke-basepack now belong in their own namespace, namely <tt>Netzke::Basepack</tt>. This prepares space for new packs of components (such as the recently started [community pack](http://github.com/netzke/netzke-communitypack)).

## Changes in the API

Let me point out the most important changes in the API. This will be useful for those who are already familiar with Netzke.

### <tt>ext_config</tt> gets deprecated

With this release quite a few things have been simplified. The most important is that I tried to make developing Netzke components as familiar as possible to those not new to Ext JS, keeping the level of extra knowledge needed for that as low as possible. Let's take an example. Before you'd need to use the <tt>ext_config</tt> config option to specify what configuration would go straight to the constructor for the JavaScript class:

~~~ruby
netzke :clerks_with_custom_bottom_bar,
    :class_name => 'GridPanel',
    :model => 'Clerk',
    :ext_config => {
      :bbar => nil,
      :tbar => [...]
    }
~~~

Now you can simply do:

~~~ruby
netzke :clerks_with_custom_bottom_bar,
    :class_name => 'GridPanel',
    :model => 'Clerk',
    :bbar => nil,
    :tbar => [...]
~~~

The idea of using <tt>ext_config</tt> was to filter out those options required for instantiating a JavaScript class. Sometimes it was pretty unclear whether a specific option should be inside of <tt>ext_config</tt> or outside of it, as in the following example:

~~~ruby
netzke :bosses_with_permissions, 
  :class_name => "GridPanel", 
  :model => 'Boss', 
  :ext_config => {
    :prohibit_update => true,
    :prohibit_delete => true
  }
~~~

Why would anyone need to remember that <tt>prohibit_update</tt> should be inside <tt>ext_config</tt>? Well, it's not needed anymore. Now _everything_ that you put into the configuration hash will by default go to the constructor of the JavaScript class, unless the component decides to filter some options out by overriding the <tt>server_side_config_options</tt> method.

So, the above example can now be written as follows:

~~~ruby
netzke :bosses_with_permissions, 
  :class_name => "GridPanel", 
  :model => 'Boss', 
  :prohibit_update => true,
  :prohibit_delete => true
~~~

### Define <tt>Ext.Container</tt>'s <tt>items</tt> directly

Another example of Netzke bringing in extra things to remember was the way to define regions in <tt>BorderLayoutPanel</tt>:

~~~ruby
netzke :grid_and_form, 
  :class_name => "BorderLayoutPanel",
  :regions => {
    :center => {
      :class_name => "GridPanel",
      :model => "Boss"
    },
    :south => {
      :class_name => "FormPanel",
      :model => "Clerk",
      :region_config => {
        :height => 150,
        :split => true
      },
      :ext_config => {          # that terrible ext_config again!
        :title => "Clerk data"
      }
    }
  }
~~~

What would be a more Ext way of doing this? Probably something like this:

~~~ruby
netzke :grid_and_form, 
  :class_name => "BorderLayoutPanel",
  :items => [{
    :region => :center,
    :class_name => "GridPanel",
    :model => "Boss"
  },{
    :region => :south,
    :height => 150,
    :split => 50,
    :class_name => "FormPanel",
    :model => "Clerk",
    :title => "Clerk data" # lot nicer!
  }]
~~~

Well, that's exactly what you can do now! And a nice extra value of it is that now you can put _anything_ into your BorderLayoutPanel, not only a Netzke component! Before it would take a painful work around to do it, and now you can simply do the following:

~~~ruby
netzke :grid_and_form,
  :class_name => "BorderLayoutPanel",
  :items => [{
    :region => :center,
    :class_name => "GridPanel",
    :model => "Boss"
  },{
    :region => :south,
    :height => 150,
    :split => 50,
    :class_name => "FormPanel",
    :model => "Clerk",
    :title => "Clerk data"
  },{
    # extra region
    :region => :east,
    :width => 300,
    :html => "I'm not a Netzke component!"
  }]
~~~

Much cleaner, isn't it? But it's even more: _any_ <tt>Ext.Container</tt>-descendant-based Netzke component can embed a Netzke component as its item (thus not only <tt>BorderLayoutPanel</tt>) - the only thing needed is to specify the <tt>class_name</tt>. Here's an example of defining a <tt>Ext.TabPanel</tt>-based component from scratch:

~~~ruby
class SimpleTabPanel < Netzke::Base
  js_base_class "Ext.TabPanel"

  js_property :active_tab, 0

  config :items => [{
            # A primitive BorderLayoutPanel here
            :class_name => "BorderLayoutPanel",
            :title => "A border layout panel",
            :items => [{
              :region => :north,
              :height => 100,
              :title => "I'm NOT a Netzke component",
              :html => "I'm a simple panel"
            },{
              :region => :center,
              :class_name => "ServerCaller"
            },{
              :region => :west,
              :width => 300,
              :split => true,
              :class_name => "ExtendedServerCaller"
            }]
        },{
          :class_name => "ExtendedServerCaller"
        }]
end
~~~

bq. This is an example taken from the test application bundled with netzke-core - I highly recommend you to study it as it provides a good deal of other examples.

The <tt>config</tt>, <tt>js_property</tt>, and <tt>js_base_class</tt> methods, while speaking for themselves, are explained in the next session.

### A set of new DSL methods

Before, when you were creating custom Netzke components, you would need to override methods like <tt>js_extend_properties</tt>, and <tt>actions</tt>, and <tt>initial_aggregatees</tt>, etc - and the resulting code wasn't too easy to read. Now you can use several handy DSL methods to make your code much more readable.

#### Setting the default configuration

You probably already know that you can configure pre-built components at the moment of embedding them into a view:

~~~html
<%= netzke :books, 
      :class_name => "Basepack::GridPanel", 
      :model => "User", 
      :title => "My Books" %>
~~~

But what if you need some of these options to be set by default in a custom component? Say, you're extending the <tt>Basepack::GridPanel</tt>, and you want your new grid to serve specifically for displaying books, and always be protected from updates. Before you'd need to override the <tt>default_config</tt> method, but now you can simply do:

~~~ruby
class BooksGrid < Netzke::Basepack::GridPanel
  config  :model => "Book",
          :title => "My Books",
          :prohibit_update => true
end
~~~

This is an example of usage of the new <tt>config</tt> DSL method. It can also accept a block (returning a hash), from which you can call other instance methods:

~~~ruby
  class BooksGrid < Netzke::Basepack::GridPanel
    config do
      {
        :model => "Book"
        :title => initial_config[:title].upcase, # using the value passed in the configuration
        :prohibit_update => true
      }
    end
  end
~~~

#### Declaring components

Another DSL method is called <tt>component</tt>, and that's what you should use instead of <tt>initial_aggregatees</tt>:

~~~ruby
component :books, :class_name => "MySpecialGrid", :model => "Book"
~~~

Then you can reference to this component when you declare the items:

~~~ruby
config :items => [{
  :books.component
}]
~~~

More details in [this blog post](http://writelesscode.com/blog/2009/09/24/building-rails-extjs-reusable-components-with-netzke-part-3/).

#### Declaring endpoints

Use the <tt>endpoint</tt> method do declare what used to be called an _API point_. Before you would do:

~~~ruby
api :tell_me_the_truth
def tell_me_the_truth(params)
  # do stuff
end
~~~

Now you can do:

~~~ruby
endpoint :tell_me_the_truth do |params|
  # do stuff
end
~~~

#### Defining actions

Use the <tt>action</tt> method to declare actions. Before you would do:

~~~ruby
def actions
  super.merge({
    :add => {:title => "Add new", :icon => "the path to the add.png icon"},
    :edit => {:title => "Modify", :icon => "the path to the edit.png icon"}
  })
end
~~~

Now you can do:

~~~ruby
action :add, :title => "Add new", :icon => :add
action :edit, :title => "Modify", :icon => :edit
~~~

(note that providing a symbol for an FamFamFam icon simply works)

Then you can reference the actions like this:

~~~ruby
config :bbar => [:add.action, :edit.action]
~~~

Or, on the _class_ level:

~~~ruby
js_property :bbar, [:add.action, :edit.action]
~~~

You may know that one can specify a pretty complex menu structure (as shown in [Ext JS examples](http://dev.sencha.com/deploy/dev/examples/toolbar/toolbars.html)), and all you need to do to say that some clickable item in that structure is bound to an action, is to provide the symbol followed with +.action+:

~~~ruby
# example taken from component_with_actions.rb from netzke-core test application
js_property :tbar, [{
  :xtype =>  'buttongroup',
  :columns => 3,
  :title => 'A group',
  :items => [{
      :text => 'Paste',
      :scale => 'large',
      :rowspan => 3, :iconCls => 'add',
      :iconAlign => 'top',
      :cls => 'x-btn-as-arrow'
  },{
      :xtype => 'splitbutton',
      :text => 'Menu Button',
      :scale => 'large',
      :rowspan => 3,
      :iconCls => 'add',
      :iconAlign => 'top',
      :arrowAlign => 'bottom',
      :menu => [:some_action.action]
  },{
      :xtype => 'splitbutton', :text => 'Cut', :menu => [:another_action.action]
  }, 
  :another_action.action,
  {
      :menu => [:some_action.action], :text => 'Format'
  }]
}]
~~~

#### Defining JavaScript properties and methods

Now you can use the <tt>js_method</tt> to define a public method in the component's JavaScript class, and <tt>js_property</tt> to define a public property (or <tt>js_properties</tt> to define several properties at once):

~~~ruby
js_method :on_edit, <<-JS
  function(){
    // do something
  }
JS
~~~

~~~ruby
js_property :title, "Some Title"
~~~

#### Defining JavaScript base class

Use the <tt>js_base_class</tt> to specify from which JavaScript class your component's JavaScript class should be inheriting. Before you would do:

~~~ruby
def js_base_class
  "Ext.TabPanel"
end
~~~

Now you can simply do:

~~~ruby
js_base_class "Ext.TabPanel"
~~~

That's it for the API changes for now. It would probably take too long to cover all the new things in details, and that's why I definitely recommend looking into the test Rails 3 application bundled with both netzke-core and netzke-basepack for more examples. Also read the [updated rdocs](http://api.netzke.org/) - I'm going to focus on documenting the code more diligently than before, so it becomes the main source of information on how to use Netzke.

## Some things left out

<img align="right" class="frame-right" width="380" height="390" src="http://writelesscode.com/images/2010-11-01-2.jpg" alt="Tweets about Netzke"/>Some lesser used functionality has been left out: 1) persistent on-the-fly configuration of components (such as resizing regions in BorderLayoutPanel, or the position/size of the Window component, etc), and 2) dynamic configuration of columns/fields in Grid/FormPanel. The way it was implemented was too much bound to specific authorization scenarios, which 1) made the code harder to understand, 2) made it difficult to use a different authorization approach (or use no authorization solution at all), 3) required ActiveRecord even for components other than those that work with data (Grid/FormPanel). I decided to get rid of those dependencies, with the intention to extract persistent configuration into a separate gem later. This is the work I have already started, and, hopefully, it'll soon provide a way to plug in any custom authorization solution implemented for the ORM of choice.

bq. Note, that Grid/FormPanel still require ActiveRecord as the _data_ layer. It'd take an extra effort to implement those as ORM-independent components, and while I haven't yet had a need for that myself, I do hope to do this one day (secretly hoping for some help).

Well, this concludes this blog post, thanks for reading it! You may follow me on [twitter](http://twitter.com/mxgrn) for more frequent updates on Netzke, along with some random findings about Ruby, JavaScript and Mac OSX. Stay tuned!
