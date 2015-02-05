--- 
title:      "Netzke-Core Update: Embedding Widgets Can't Be Easier"
created_at: 2010-02-21
kind: article
outdated: true
tags: [tutorials, netzke]
excerpt: Empedding Netzke components into a Rails application.
--- 
The Netzke project is now over a year old, and several developers have since then started using it in their projects, creating their own widgets and using the pre-built widgets from [netzke-basepack](http://github.com/netzke/netzke-basepack). Recently, the [netzke-core](http://github.com/nomadcoder/netzke-core) gem has been updated to version 0.5.0, and this update has simplified the way a Netzke widget gets embedded in your Rails application. First of all, no longer is it needed to define widgets in the controller - simply put them straight into your views using the <tt>netzke</tt> helper. All the AJAX communication between the client and server side of widgets will now be routed through the <tt>netzke_controller</tt>, which is a part of netzke-core. In this post I'll provide a couple of examples of embedding Netzke widgets into the Rails views. In case you haven't yet appreciated the simplicity and flexibility of the Netzke framework, then, hopefully, this post will get you hooked.

But first, let's see what's improved in the way we prepare our Rails application for using Netzke. In the layout, within the <tt>header</tt> tag, instead of all the previously used netzke-related helpers and includes of ExtJS, you may simply put <tt>netzke_init</tt>:

<% highlight :ruby do %>
<%%= netzke_init %>
<% end %>    

This will include all the JavaScript and stylesheets needed to run ExtJS and Netzke, as well as provide for <tt>Ext.onReady</tt> call to render the widgets after the page load.

Provided you have the <tt>map.netzke</tt> line in your <tt>config/routes.rb</tt>, you may now directly include Netzke widgets in your views.

## Grid
Using Netzke is probably the easiest and most flexible way of integrating a full featured Ext.grid.EditorGridPanel-based grid into a Rails view. See it as scaffolding with lots of extras (such as on-the-fly reconfiguration of your grid). The code in its simplest form looks like this:

<% highlight :ruby do %>
<%%= netzke :grid_panel, :model => "Boss" %>
<% end %>    

In the need of deviating from the defaults, Netzke's GridPanel is very configurable. Just go [here](http://netzke-demo.writelesscode.com/grid_panel) for live demo and comprehensive examples.

## Form
Netzke's FormPanel is almost as intelligent as GridPanel, in the sense that it provides for great defaults for the form fields, as well as flexible overrides and dynamic reconfiguration:

<% highlight :ruby do %>
<%%= netzke :form_panel, :model => "Boss", :record_id => Boss.first.id %>
<% end %>    

In fact, if you have played around with Netzke's GridPanel, you've probably seen FormPanel in action, when editing records in the form, doing the search, doing the dynamic reconfiguration, etc.

## Tab panel
Netzke's TabPanel (as well as AccordionPanel below) is an example of a widget that provides for dead-simple embedding of other widgets (with or without dynamic loading):

<% highlight :ruby do %>
<%%= netzke :tab_panel, :items => [{
  :class_name => "GridPanel", :model => "Boss"
}, {
  :class_name => "GridPanel", :model => "Clerk"
}] %>
<% end %>

## Accordion panel
The AccordionPanel has similar configuration to the TabPanel:

<% highlight :ruby do %>
<%%= netzke :accordion_panel, :items => [{
  :class_name => "GridPanel", :model => "Boss"
}, {
  :class_name => "GridPanel", :model => "Clerk"
}] %>
<% end %>    

## A panel with the "border" layout
This widget I use very often for creating custom complex widgets, where different nested widgets interact in a certain way (see several tutorials on this blog). The example below simply provides a panel with 3 nested widgets:

<% highlight :ruby do %>
<%%= netzke :border_layout_panel, :regions => {
  :center => {
    :class_name => "GridPanel", :model => "Boss"
  },
  :south => {
    :class_name => "Panel", 
    :ext_config => {:html => "Just a panel with text here"},
    :region_config => {:height => 100}
  },
  :east => {
    :class_name => "FormPanel", :model => "Boss",
    :region_config => {:width => 300}
  }
} %>
<% end %>    

## Window
Recent update to Netzke made it possible to declare several Ext.Window-based widgets on a page: when a window is triggered shown, it dynamically loads the embedded widget. Here's an example of a window that embed a previously created complex widget (see the corresponding [tutorial](http://writelesscode.com/blog/2009/09/24/building-rails-extjs-reusable-components-with-netzke-part-3/)):

<% highlight :ruby do %>
<%%= netzke :window, :item => {
  :class_name => "BossesAndClerks"
} %>
<% end %>    

See Netzke's window in action on the [netzke-demo page](http://netzke-demo.writelesscode.com/window).

## Conclusion
So, could it be simpler than this? The examples above only give you a glance of what's possible. You may realize, for example, that Netzke allows for unlimited nesting of widgets (which wouldn't be possible without the powerful ExtJS architecture), each of which will "talk" to its own server side instance with minimum (or none, as in these examples) effort from the developer. If it doesn't sound a bit of a magic to you, then you've possibly not been working on complex Rails/ExtJS apps long enough :)

## Future
I realize that Netzke badly needs documentation and guides on developing new widgets. In mean while several [brave people](http://groups.google.com/group/netzke) started using Netzke in their projects, going through reading Netzke code and the [tutorials](http://writelesscode.com). They also have helped me significantly improve Netzke (thanks guys!). I plan on starting a community driven Netzke guides online project, which would allow to easily contribute a tutorial or two.

Besides that, Rails 3 is coming very soon, and compatibility branches of the Netzke-related projects will soon appear on the GitHub. Another idea is to set up a repository of community-contributed widgets (yes, there are a couple of them at the moment), which would not only be a source of different pre-built widget, but also an example of other developers' code.

As a very good starting point for getting into Netzke, many have chosen to clone the [netzke-demo](http://github.com/netzke/netzke-demo) project and play with it in their development environment. Don't hesitate to join the Netzke Google Groups, subscribe to this blog, and/or follow me on [Twitter](http://twitter.com/mxgrn).
