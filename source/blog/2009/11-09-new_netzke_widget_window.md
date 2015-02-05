--- 
title:      "New Netzke Widget: Window"
created_at: 2009-11-09
kind: article
outdated: true
tags: [tutorials, netzke]
excerpt: In-depth view on how to create a Window Netzke component.
--- 
<em>UPDATE (2011-11-11): This tutorial is outdated with the release of netzke-basepack v0.5.0. Please, refer to <a href="http://wiki.github.com/netzke/netzke">the project's wiki</a> for updated articles.</em> Working on my client's app (the first commercial Netzke-driven application planned to go live on December 1st), I got tired of repetitively writing JavaScript/ExtJS code for different windows that pop up here and there. They were all doing a similar thing: dynamically loading some Netzke widget at the moment of being shown. Besides boring coding, it had another drawback: the windows didn't remember its dimensions and position. So, finally I asked myself: how long would it take to write an Ext.Window-based Netzke widget that would be able to aggregate other widgets just the same way as BorderLayoutPanel, TabPanel, and others do? Well, it appeared to be so simple, that I'll dare post the whole source code in this post. For those who have followed my previous tutorials, it's not going to be difficult to understand what's going on here.

<% highlight :ruby do %>
module Netzke
  # == Window
  # Ext.Window-based widget
  # 
  # == Features
  # * Persistent position
  # * Persistent dimensions
  # 
  # == Instance configuration
  # <tt>:height</tt> and <tt>:width</tt> - besides accepting a number,
  # can accept a string specifying relative sizes, 
  # calculated from current browser window dimensions.
  # E.g.: :height => "90%", :width => "60%"
  class Window < Base
    # Based on Ext.Window, naturally
    def self.js_base_class
      "Ext.Window"
    end
    
    # Set the passed item as the only aggregatee
    def initial_aggregatees
      {:item => config[:item]}
    end
    
    # Extends the JavaScript class
    def self.js_extend_properties
      {
        :layout => "fit",
        :init_component => <<-END_OF_JAVASCRIPT.l,
          function(){
            // Width and height may be specified as percentage of available space, e.g. "60%".
            // Convert them into actual pixels.
            Ext.each(["width", "length"], function(k){
              if (Ext.isString(this[k])) {
                this[k] = Ext.lib.Dom.getViewHeight() * 
                  parseFloat("." + this[k].substr(0, this[k].length - 1)); // "66%" => ".66"
              }
            });
            
            // Superclass' initComponent
            #{js_full_class_name}.superclass.initComponent.call(this);
            
            // Set the move and resize events after window is shown, 
            // so that they don't fire at initial rendering
            this.on("show", function(){
              this.on("move", this.onMove, this);
              // Work around firing "resize" event twice (currently a bug in ExtJS)
              this.on("resize", this.onSelfResize, this, {buffer: 50});
            }, this);
            this.instantiateChild(this.itemConfig);
          }
        END_OF_JAVASCRIPT
      
        :on_move => <<-END_OF_JAVASCRIPT.l,
          function(w,x,y){
            this.moveToPosition({x:x, y:y});
          }
        END_OF_JAVASCRIPT

        :on_self_resize => <<-END_OF_JAVASCRIPT.l,
          function(w, width, height){
            this.selfResize({w:width, h:height});
          }
        END_OF_JAVASCRIPT
      }
    end
    
    # Processing API calls from client
    api :move_to_position
    def move_to_position(params)
      update_persistent_ext_config(:x => params[:x].to_i, :y => params[:y].to_i)
      {}
    end
    
    api :self_resize
    def self_resize(params)
      update_persistent_ext_config(:width => params[:w].to_i, :height => params[:h].to_i)
      {}
    end
  end
end
<% end %>

The way you declare the widget is as simple as this:

<% highlight :ruby do %>
netzke :my_window, 
       :widget_class_name => "Window", 
       :item => {
         :widget_class_name => "AnyNetzkeWidget", 
         ... # aggregated widget configuration
       },
       :ext_config => {
         :width => 200,
         :hight => 100,
         :x => 50,
         :y => 50,
         ...
       }
<% end %>

Of course, as always, in <tt>ext_config</tt> you may add any other configuration options that are known by Ext.Window. But there's a little sugar from Netzke: you may specify dimensions - both width and *height* - in percentage of available browser window dimensions: e.g.

<% highlight :ruby do %>
:width => "90%"
:width => "80%"
<% end %>

In order to use this new widget, you'll need to update to the latest Netzke gems (sudo gem update netzke-basepack). I needed to slightly modify netzke-core, so that it would correctly handle Ext.Window-based widgets, as it's not a "classical" widget that is always rendered inside of a "fit" layout, or in a div. Ext.Window doesn't need any containers, as it dynamically creates its own one when, at the moment its <tt>show</tt> method gets called.

For the sake of quick experiment, I replaced one of the windows shown by basepack's GridPanel with the new Window widget, so that you can see it in action at [live Netzke demo](http://netzke-demo.writelesscode.com/basic_app/demo). Open any view which contains a grid, and press "Add in form" button - and here it comes. Move it around, resize it, and find it back some other day just the way you left it: same dimensions, same position. Slowly I'll replace all windows in [netzke-basepack](http://github.com/netzke/netzke-basepack) with this one, which will make the code yet DRYer, and user's experience with Netzke yet more consistent.
