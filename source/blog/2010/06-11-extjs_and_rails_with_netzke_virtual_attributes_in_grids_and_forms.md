---
title:      "ExtJS and Rails with Netzke: Virtual Attributes in Grids and Forms"
created_at: 2010-06-11
kind: article
outdated: true
tags: [tutorials, netzke, 0-7]
excerpt: This blogpost will focus on how Netzke lets you specify and configure a model's (virtual) attributes mapped to columns (in grids) and fields (in forms).
---
<img align="right" class="frame-right" width="314" height="182" src="http://writelesscode.com/images/2010-06-11.jpg" alt="ExtJS and Rails With Netzke. Virtual attributes."/> This blogpost will focus on how Netzke lets you specify and configure a model's (virtual) attributes mapped to columns (in grids) and fields (in forms). The way it was done before passed through several iterations of improvements, and finally I can say that I found an approach I'm currently happy with. It's very flexible, but at the same time the resulting code is readable, concise, and simply looks very habitual. Which problems is it solving? 1) Imagine, you want to render some HTML code in a grid cell (a link, an icon, etc). You could easily declare a method right in the model and use it as a virtual attribute. But it totally doesn't feel right to put this into your model, as it's about presenting the data, not the data itself. 2) Besides, different widgets may need different "representational" attributes for the same model - how would we handle that? Mix them in, in some way, into the model all together? Doesn't feel right either, especially if there are naming conflicts. 3) Also, different widgets may need to render the same attribute in a different way (e.g., with different formatting). Well, all this is now covered in Netzke. Read on.

**Update**: This tutorial is _outdated._ See a better way of achieving the same thing "here":http://demo.netzke.org/grid_panel (the "Associations, Rails validations, "virtual" columns, and more" section).

h3. Put it in the model, you're the boss

Netzke lets you as a developer decide, where a specific virtual attribute belongs. Is it generic for a model? Can it be reused in different views or in business logic calculations? Fine, then just declare it in the model:

~~~ruby
  class Clerk < ActiveRecord::Base
    # ...

    # a virtual attribute
    def name
      "#{last_name}, #{first_name}"
    end

    # another virtual attribute
    def updated
      updated_at > 5.minutes.ago
    end
  end
~~~

bq. Check out the basics of virtual attributes in Ryan Bates' screencast "here":http://railscasts.com/episodes/16-virtual-attributes.

And now we need to let grids and forms know that this is a virtual attribute (otherwise how can it be distinguished from a generic instance method?)

~~~ruby
  class Clerk < ActiveRecord::Base
    # ...

    # a virtual attribute
    netzke_attribute :name
    def name
      "#{last_name}, #{first_name}"
    end

    # another virtual attribute
    netzke_attribute :updated
    def updated
      updated_at > 5.minutes.ago
    end
  end
~~~

Perfect. This way Netzke grids and forms will pick those 2 up, just as the "real" attributes. But hey, the "updated" attribute will be rendered as "true" or "false" - not very much visually stunning. Why won't we try to display a little icon instead, a switched on bulb when "updated" returns "true", and a switched off one when it's "false"? Here's where we need to decide if such code really belongs in the model. And the answer is "no, it doesn't". Besides, what if in forms we're fine with just "true" and "false", and some other grid should display it as colored flags instead of bulbs?

h3. Conventions, sweet conventions

A convention to rescue. Any GridPanel and FormPanel from Netzke will know, that if there's a class named in a certain way, this class will be used as the model to deliver the data, *not* the originally specified class. The convention is: <model_name>For<widget_class_name>. And it should be in Netzke::ModelExtensions module (put the class file into <tt>lib/netzke/model_extensions</tt>), e.g.:

~~~ruby
module Netzke::ModelExtensions
  class ClerkForGridPanel < Clerk

    # Virtual attribute defined below
    netzke_attribute :updated_bulb

    def updated_bulb
      bulb = updated ? "on" : "off"
      "<div class='bulb-#{bulb}' />"
    end
  end
end
~~~

Two things to note here. First of all, the name of the class: <tt>ClerkForGridPanel</tt>. It means it will be effective for any widget of class GridPanel bound to the model called Clerk. Second, this class is inherited from the model itself (<tt>Clerk</tt>). Naturally, it'll have access to all the attributes of the original class (such as <tt>updated</tt>).

bq. Another example may be handy here. Imagine, you have a widget that extends GridPanel, and it's called "MyCoolGrid" (maybe it implements a button that allows deleting all the data from the model's table?) Let's say it should sometimes display the data for Clerks, and sometimes - for Bosses. You can define virtual attributes separately for those cases, in classes <tt>ClerkForMyCoolGrid</tt> and <tt>BossForMyCoolGrid</tt> respectively, no conflicts whatsoever.

I hope, by now you're getting the idea about how virtual attributes are handled in Netzke. But that's not all.

h3. Preconfiguring "real" attributes

The same method, <tt>netzke_attribute</tt>, can be used to preconfigure the way the attribute's value should be displayed in grids and forms. E.g.:

~~~ruby
module Netzke::ModelExtensions
  class ClerkForGridPanel < Clerk

    # Make the column look nice
    netzke_attribute :updated_bulb,
                      :width => 40,
                      :label => "<div class='bulb-off' />",
                      :tooltip => "Recently updated"

    # Preconfigure a "real" attribute
    netzke_attribute :salary,
                      :renderer => "usMoney",
                      :label => "$",
                      :read_only => true,
                      :tooltip => "Poor fellow's wage"

    # ...
  end
end
~~~

The "salary" attribute is "real", i.e. there's a field "salary" in the "clerks" table, while "updated_bulb" is virtual (Netzke doesn't care). Because it's clear, that <tt>ClerkForGridPanel</tt> configures attributes for a <tt>GridPanel</tt>, we can easily provide additional configuration for each Ext *column* that will be bound to a specific attribute. Setting the width? The renderer? The tooltip? The header? Almost any valid option for "Ext.grid.ColumnModel":http://www.extjs.com/deploy/dev/docs/?class=Ext.grid.ColumnModel will do.

Similarly the <tt>FormPanel</tt> (e.c. in the class called <tt>BossForFormPanel</tt>): any option for of "Ext.form.Field":http://www.extjs.com/deploy/dev/docs/?class=Ext.form.Field - based component (such as "xtype", "inputType", etc) can be provided. Simple, clean, and powerful.

h3. One last thing

You noticed, that you may use the <tt>netzke_attribute</tt> method both in the model class, as in the extension class. There 2 more methods from Netzke to let you control which attributes should be picked up by widgets like GridPanel and FormPanel by default, and you can put them in both places, too:

* <tt>netzke_expose_attributes</tt> lets you be specific about the attributes you want to be accessible to Netzke, and their order
* <tt>netzke_exclude_attributes</tt> does just the opposite

That's it. See all the provided examples (and more) in action on GridPanel "live demo":http://netzke-demo.writelesscode.com/grid_panel, or in a BasicApp "live demo":http://netzke-demo.writelesscode.com/basic_app .
