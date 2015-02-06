---
title:      Better Separation of Concerns in Netzke Core with Ext.Direct
created_at: 2011-03-01
kind: article
tags: [tutorials, netzke, 0-7]
excerpt: This blog post shows off a very cool new feature in Netzke Core which allows for much cleaner code in composite components.
---
<img align="right" class="frame-right" width="225" height="225" src="http://writelesscode.com/images/2011-03-01-1.jpeg" alt="Let's tweak this Sencha Ruby on Rails component framework a bit!"/>In Russia, spring conventionally started today, on the 1st of March (the rest of the world may need to wait until the 21st), so, let me celebrate this with a new blog post about a nice feature that found its way into the recent 0.6.6 release of [Netzke Core](https://github.com/netzke/netzke-core). Among other improvements, it incorporates some [great work](http://pschyska.blogspot.com/2011/02/introducing-extdirect-to-netzke.html) of [Paul Schyska](http://twitter.com/pschyska): he re-implemented the components client-server protocol using [Ext.Direct](http://www.sencha.com/products/extjs/extdirect/). The main result of it are multiplexed remote calls between the client and the server side of a component. Not only consecutive remote calls from a component get wrapped-up in a single actual call, but also calls from different child components in a composite component, being fired in a row, will be multiplexed into a single call to the server! The response from the server will carry back personal responses for each of the components participating in the communication, and for each individual call. This tutorial will show how we can benefit from this feature, making our code even more modular.

To illustrate multiplexing of remote calls, we'll update the BossesAndClerks component we [created before](http://writelesscode.com/blog/2009/09/24/building-rails-extjs-reusable-components-with-netzke-part-3/). This component is a part of the [Netzke Demo](https://github.com/netzke/netzke-demo) project, and by the time you're reading this, it's been already [updated](http://demo.netzke.org/#bosses_and_clerks) according to this tutorial. But I have created a gist with the [original component's code](https://gist.github.com/847722) so that we can refer to it here.

## Old way
I'll remind you how this component works. When the user selects a boss in the upper grid, 2 things happen: 1) the lower grid gets updated with the clerks assigned (through one-to-many association) to the selected boss, and 2) the info panel on the right gets updated with some statistics on the selected boss as well. In order to achieve this with one call to the server, we used to implement the `select_boss` endpoint of our component in such a way, that it updates both the clerk grid as well as the info panel. It implied that our top component "knew" how the clerk grid is supposed to work - after instantiating the clerks grid component (line 58 in the gist), we had to call its `get_data` to get the records, and then call the clerks panel's JavaScript-side method `loadStoreData` to display those records.

<img align="right" class="frame-right" width="226" height="223" src="http://writelesscode.com/images/2011-03-01-2.jpeg" alt="Mind your business"/>While it's normally not a big sin to design a top component in the way that it "knows" *something* about its children, even in this simple example you clearly feel that updating the clerks grid should be this grid's *own business*. All what the top component, ideally, should do, is to command the clerks grid to update itself. I'll give you a quick example of a problem which would be hard to solve without writing clumsy code, should we follow the old approach: say, you want the clerks panel to show the update mask (it does, btw, if and when updating itself!). Should the top component take care of it *as well*? It can become so bad so quickly! We need a better [separation of concerns](http://en.wikipedia.org/wiki/Separation_of_concerns).

And that's where Paul's contribution comes very, very handy. I can't wait to update our example!

## New way
For the sake of even better illustration (and cleaner code), we'll extract the info panel into a separate simple component, capable of updating itself (an update mask would be very welcome also here!). Let's call it `BossDetails`:

~~~ruby
class BossDetails < Netzke::Basepack::Panel
  js_property :padding, 5

  js_property :title, "Info"

  js_method :update_stats, <<-JS
    function(){
      // Create and show the mask
      if (!this.maskCmp) this.maskCmp = new Ext.LoadMask(this.getEl(), {msg: "Updating..."});
      this.maskCmp.show();

      // Call endpoint
      this.update({}, function(){
        // Hide mask (we're in the callback function)
        this.maskCmp.hide();
      }, this);
    }
  JS

 endpoint :update do | params |
    # updateBodyHtml is a JS-side method we inherit from Netkze::Basepack::Panel
    {:update_body_html => body_content(boss), :set_title => boss.name}
  end

  # HTML template used to display the stats
  def body_content(boss)
    %Q(
      <h1>Statistics on clerks</h1>
      Number: #{boss.clerks.count}<br/>
      With salary > $5,000: #{boss.clerks.where(:salary.gt => 5000).count}<br/>
      To lay off: #{boss.clerks.where(:subject_to_lay_off => true).count}
    ) if boss
  end

  private
    def boss
     @boss |  | = config[:boss_id] && Boss.find(config[:boss_id]) |
    end
end
~~~


As you may see, the component gets a JavaScript method called "updateStats", which, in its turn, makes the call to the endpoint, as well as handles the load mask. You may also notice, that for the endpoint to respond with the HTML for a specific boss, we need to configure this component with `boss_id`. Just as in the case with the clerks grid (line 66 of the gist), we get `boss_id` from our top component's session:

~~~ruby
component :boss_stats do
  {
    :class_name => "BossDetails",
    :boss_id => component_session[:selected_boss_id]
  }
end
~~~

The responsibility of our top component, in respect to its children, has been cut down to setting `component_session[:selected_boss_id]` to the proper value when the user selects the boss:

~~~ruby
endpoint :select_boss do | params |
  component_session[:selected_boss_id] = params[:boss_id]
end
~~~

Yay, no more shaman dances around children components here!

Now all what's left, is update our client-side event on selecting a row in the bosses grid (in initComponent):

~~~javascript
this.getChildComponent("bosses").on('rowclick', function(self, rowIndex){
  // The beauty of using Ext.Direct: calling 3 endpoints in a row, which results in a single call to the server!
  this.selectBoss({boss_id: self.store.getAt(rowIndex).get('id')});
  this.getChildComponent('clerks').getStore().reload();
  this.getChildComponent('boss_stats').updateStats();
}, this);
~~~

## Extra thought and remarks

This is all very cool, you may think, but wouldn't this all work before - before Netzke got that Ext.Direct power? Well, it might, indeed! But then it would result in three - not one - parallel requests to the server!

Then, you may have noticed that the old version of BossesAndClerks was also updating the title for the clerks grid (line 61). How should we do it now? This is where you, as developer, will make the decision. One way would be to do just as before:

~~~ruby
endpoint :select_boss do | params |
  component_session[:selected_boss_id] = params[:boss_id]
  {:clerks => {:set_title => "..."}}
end
~~~

... and it would still work perfectly. Another way - if you want to incapsulate this behavior into the clerks grid as a separate component - is, well, to create such a custom component. Inherit it from `Netzke::Basepack::GridPanel`, do all the work inside of it, and then use it instead of GridPanel in our top component:

~~~ruby
component :clerks do
  {
    :class_name => "MyCustomClerksGrid",
    :boss_id => component_session[:selected_boss_id]
  }
end
~~~

And that's it for this time. I hope, by now you see the power of this inner enhancement to Netzke Core, and let's show Paul our appreciation by [following him on twitter](http://twitter.com/pschyska).
