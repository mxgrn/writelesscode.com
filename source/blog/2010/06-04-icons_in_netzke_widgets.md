--- 
title:      Icons in Netzke Widgets
created_at: 2010-06-04
kind: article
outdated: true
tags: [tutorials, netzke]
excerpt: How to specify custom icons in Netzke components.
--- 
<img align="right" class="frame-right" src="http://writelesscode.com/images/2010-06-04.jpg" alt="ExtJS and Rails With Netzke. Custom Widget Actions."/>
Recent update to [Netzke](http://github.com/netzke/netzke-basepack) (the framework that helps you build ExtJS/Ruby-on-Rails applications) introduces support for [FamFamFam Silk icons](http://www.famfamfam.com/archive/silk-icons-thats-your-lot/). All you have to do is download the icons and put them into your <tt>public/images/icons</tt> folder (so that the icons are accessible via <tt>http://yourhost.com/images/icons</tt>) - the next time you restart your Rails application, Netzke will detect the icons and start using them. As always, this update is reflected on the Netzke [live demo](http://netzke-demo.writelesscode.com/) page. And if you want to know how to customize the icons - read on.

Let's see how we can change the icons for the GridPanel. Inherit your customized grid panel from Netzke::GridPanel (see [this tutorial](http://blog.local/blog/2009/10/05/extjs-and-rails-with-netzke-custom-widget-actions.html) for an example), and override the <tt>actions</tt> method, specifying custom icons for any of the available actions, e.g.:

<% highlight :ruby do %>
module Netzke
  class Books < GridPanel
    def actions
      super.deep_merge({
        :add => {:icon => "/images/icons/book_add.png"},
        :edit => {:icon => "/images/icons/book_edit.png"},
        :del => {:icon => "/images/icons/book_delete.png"}
      })
    end
  end
end
<% end %>

Icons location in the application is configured by setting <tt>Netzke::Base.config[:icons_uri]</tt> to the relative URL to the icons (defaults to "/images/icons"). For example (at the end of your environment.rb):

<% highlight :ruby do %>
Netzke::Base.config[:icons_uri] = "/images/iconz/"
<% end %>

Also note, that the <tt>icon</tt> option for an action is not specific to Netzke, it's just a valid configuration option for Ext.Action.
