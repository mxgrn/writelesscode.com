---
title:      Sencha Touch/Rails Components With Netzke, Part 1
created_at: 2011-01-13
kind: article
outdated: true
tags: [tutorials, netzke]
excerpt: This tutorial shows how to build reusable Sencha Touch components with Netzke Core.
---
<img align="right" class="frame-right" width="320" height="167" src="http://writelesscode.com/images/2011-01-12-4.jpg" alt="Sencha Touch Tutorial Result"/>**Update: Netzke currently dropped support for Sencha Touch**.

The upcoming 0.6.5 release of Netzke Core brings support for Sencha Touch, which means that you can create client-server components for mobile devices with the same ease as you do for desktop with Ext JS. Currently, there are no pre-built components for Sencha Touch in Netzke, but this tutorial is meant to show you how to get started with building them from scratch. We'll create a simple [Ext.List](http://dev.sencha.com/deploy/touch/docs/?class=Ext.List)-based Sencha Touch component to display data from a given ActiveRecord model. Then I will show you how easy it is to reuse it, by creating an [Ext.TabPanel](http://dev.sencha.com/deploy/touch/docs/?class=Ext.TabPanel)-based component that will aggregate 2 instances of the new component.

### Starting with the component
For the most impatient, here's the link to see the resulting component in action: [netzke-demo](http://demo.netzke.org/components/touch/SimpleTabPanel) (*use your mobile device or a WebKit-powered browser, such as Safari or Chrome, in order to see it*). Let's see how we get there step-by-step.

We'll do our work in the context of the [netzke-demo project](https://github.com/netzke/netzke-demo), where we already have a couple of Ruby on Rails ActiveRecord models that we used in our previous tutorials. Let's start with creating a component called `SimpleList` (in <tt>app/components/touch/simple_list.rb</tt>):

~~~ruby
module Touch
  class SimpleList < Netzke::Base
    js_base_class "Ext.List"
  end
end
~~~

As you can see, the JavaScript class of our component will be extending `Ext.List`.

If now we try to load the component now, we wouldn't see anything, because `Ext.List` requires some configuration, which can be done in its `initComponent` method.

### Overriding `initComponent`
We could override the `initComponent` method straight in our Ruby class like this:

~~~ruby
js_method :init_component, <<-JS
  function(){
    // do our stuff
    Netzke.classes.Touch.SimpleList.superclass.initComponent.call(this);
  }
JS
~~~

But because our `initComponent` method will not need to change, and because I like to keep static JavaScript code separated from Ruby code, I will show you a simple technique of using the `js_mixin` method being introduced in Netzke Core 0.6.5. The idea of this method is that it allows you to "mixin" JavaScript objects declared in separate <tt>.js</tt>-files:

~~~ruby
module Touch
  class SimpleList < Netzke::Base
    js_base_class "Ext.List"

    js_mixin :main
  end
end
~~~

This will, by convention, try to read a file named <tt>app/components/touch/simple_list/javascripts/main.js</tt>, where we'll define our `initComponent`:

~~~javascript
{
  initComponent: function() {
      Ext.regModel(this.model, {
          fields: this.attrs
      });

      var sortAttr = this.sortAttr;

      this.store = new Ext.data.JsonStore({
          model  : this.model,
          sorters: sortAttr,

          getGroupString : function(record) {
              return record.get(sortAttr)[0];
          },

          data: this.data
      });

      Netzke.classes.Touch.SimpleList.superclass.initComponent.call(this);
  }
}
~~~

It's a very simple setup for the `Ext.List`, where we register a model and a data store.
But what are `this.model`, `this.sortAttr`, `this.attrs`, and `this.data`? Where do they come from? They are our custom configuration options that we'll be passing to our JavaScript class's constructor from the Ruby class.

### Passing data from the Ruby class

Passing data to the constructor of the JavaScript class can be done in a couple of different ways, one of which is overriding the `configuration` method in the Ruby class (where `super` will provide us with the configuration options passed by the user of the component - this way we'll be able to know, for example, what data model our component should be bound to):

~~~ruby
module Touch
  class SimpleList < Netzke::Base
    js_base_class "Ext.List"

    js_mixin :main

    def configuration
      super.merge(
        :model => ...
        :sort_attr => ...
        :attrs => ...
        :data => ...
      )
    end

  end
end
~~~

But first let's make a step back and think about how exactly we want to be able to configure our component.

### Ruby-level configuration for our component

Because it's a very simple component, we'll make it to obey the following few configuration options:

* `model` - the name of the ActiveRecord model class from which we will fetch our data, e.g. "Boss"
* `item_tpl` - this is a native Ext.List configuration option, which specifies what will be displayed in each raw of the list, for example, `"{last_name}, ${salary}"`
* `sort_attr` - attribute of the ActiveRecord model that will be used for sorting and grouping of the records in the list (will default to the first attribute used in `item_tpl`)

These options are accessible via the `super` call of our `configuration` methods. Having this in mind, let's write the final implementation:

~~~ruby
module Touch
  class SimpleList < Netzke::Base
    js_base_class "Ext.List"

    js_mixin :main

    def configuration
      sup = super

      # extract model attributes that participate in the template
      attrs = extract_attrs(sup)

      # model's class
      data_class = sup[:model].constantize

      sup.merge(
        :data => data_class.all.map{ |r| attrs.inject({}){|hsh,a| hsh.merge(a.to_sym => r.send(a))}},
        :attrs => attrs.map{|a| a.camelize(:lower)}, # convert camelcase (more natural in JavaScript)
        :item_tpl => sup[:item_tpl].gsub(/\{(\w+)\}/){|m| m.camelize(:lower)}, # same here
        :sort_attr => (sup[:sort_attr] || attrs.first).to_s.camelize(:lower) # ... and here
      )
    end

    protected
      # Extracts names of the attributes from the temalpate, e.g.:
      # "{last_name}, ${salary}" =>
      # ["last_name", "salary"]
      def extract_attrs(config)
        config[:item_tpl].scan(/\{(\w+)\}/).map{ |m| m.first }
      end
  end
end
~~~

So, if now we embedded our component in a Rails view:

    <%%=  netzke :simple_list, :model => "Clerk", :item_tpl => "{last_name}" %>

... then we would see something like this:

![SimpleList - Clerks](/images/2011-01-12.jpg)

Congratulations, we have just finished a simple, data-driven component for Sencha Touch!

At this moment you could ask: but was it really worth it? Why taking the pain of using Netzke and wrap it all into a component, while we could just go a traditional Rails way, with views and controllers? Well, I will argue that it _was_ worth it. The 4 main advantages of using components in your application are: reusability, composability, extensibility, and encapsulation. The last one means that once a component is written, you don't need to know and care about its internals (you don't even need to know JavaScript!) in order to use it. Extensibility we'll address in the following parts of this tutorial, but now let's talk about reusability and composability.

### Reusability

You have surely already guessed that reusing the created component throughout our application would be really easy. Want to display a different model in a different view? No problem:

    <%%=  netzke :simple_list, :model => "Boss", :item_tpl => "{last_name}, ${salary}" %>

Here's the result:

![SimpleList - Bosses](/images/2011-01-12-2.jpg)

### Composability

How difficult would it be to combine 2 components into one, say, `Ext.TabPanel`? *Very* easy with Netzke:

~~~ruby
module Touch
  class SimpleList < Netzke::Base
    js_base_class "Ext.List"

    js_mixin :main

    # What this method returns is going to be used as configuration for the
    # JavaScript instance of our component (thus, accessible in the `initComponent` method)
    def configuration
      sup = super

      # extract model attributes that participate in the list
      attrs = extract_attrs(sup)

      # model's class
      data_class = sup[:model].constantize

      sup.merge(
        # For simplicity, we'll be fetching all records
        :data => data_class.all.map{ |r| attrs.inject({}){|hsh,a| hsh.merge(a.to_sym => r.send(a))}},

        # While not necessary, I like to camelize the attribute names so that they read
        # more naturally in JavaScript
        :attrs => attrs.map{|a| a.camelize(:lower)},

        # Same here: "Name: {last_name}" => "Name: {lastName}"
        :item_tpl => sup[:item_tpl].gsub(/\{(\w+)\}/){|m| m.camelize(:lower)},

        # Attribute used for sorting/grouping (defaults to the first found in the template)
        :sort_attr => (sup[:sort_attr] || attrs.first).to_s.camelize(:lower)
      )
    end

    protected
      # Extracts names of the attributes from the temalpate, e.g.:
      # "{last_name}, ${salary}" =>
      # ["last_name", "salary"]
      def extract_attrs(config)
        config[:item_tpl].scan(/\{(\w+)\}/).map{ |m| m.first }
      end
  end
end
~~~

The code is so straight forward, that it speaks for itself. And here's the result:

![SimpleList - TabPanel](/images/2011-01-12-3.jpg)

### Conclusion

This concludes the first part of the tutorial. If you're up to a complex mobile application and feel that could benefit from the modular approach, check out [Netzke](http://netzke.org), which has proven to be extremely helpful to many developers doing Ruby on Rails / Sencha Ext JS development, and now is also giving you the tools to build Sencha Touch components using the familiar API.
