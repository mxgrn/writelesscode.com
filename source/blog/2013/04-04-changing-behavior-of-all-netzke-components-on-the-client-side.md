---
title: "Changing behavior of all Netzke components on the client side"
created_at: 2013-04-04
kind: article
tags: [netzke, tutorials, 0-8]
excerpt: Here I'll quickly explain how you can modify the behavior of each and every Netzke component at the client side.
---
Netzke development is going as strong as ever, with some great support from the [community](http://groups.google.com/group/netzke/) - in the form of skillful pull requests, interesting suggestions, and joint experiments. Thanks everyone who's been helping moving it forward! Besides, I'm going to present Netzke at [RubyKaigi](http://rubykaigi.org/2013) in Tokyo, as well as a few other exciting things meant to give Netzke some exposure, are pending. You may want to follow Netzke on [Twitter](https://twitter.com/netzke) if you want to stay up-to-date with that news that may not find its way to a blog post.

In meanwhile, in this little tutorial I want to share a tip on how to extend each and every Netzke client-side class, and I'll do that on an example of implementing a custom feedback functionality. As you may know, each Netzke component, at the client side, has a method called `netzkeFeedback`, which by default displays a little box on the top of the screen:

![feedback](/images/2013-04-03-1.png)

You may already know that by overriding this method in your component, you can replace the default implementation with whatever you find appropriate - *for that specific component*. But what if you want to change the way feedback is displayed for *all* Netzke components at once?

The way to do that is to override the `netzkeFeedback` method in `Netzke.classes.Core.Mixin`, which gets mixed-in into each Netzke component. However, this has to be done *before* any Netzke component class is defined, and for that we can use `Netzke::Core.ext_javascripts` config option, which other Netzke gems (such as netzke-basepack) use to inject custom JavaScript code (which can be a patch for Ext JS or code shared by multiple components).

A good place to configure `Netzke::Core` is in `config/initializers` (e.g. in a file named `netzke.rb`):

~~~ruby
Netzke::Core.setup do |config|
  config.ext_javascripts << "#{File.dirname(__FILE__)}/javascripts/netzke_extensions.js"
end
~~~

This will instruct Netzke to load `config/initializers/javascripts/netzke_extensions.js` along with other initial JavaScript code when a Rails view with Netzke components gets loaded. As I said before, in that file we will override `netzkeFeedback`:

~~~javascript
Ext.define(null, {
  override: "Netzke.classes.Core.Mixin",
  netzkeFeedback: function(msg, options) {
    alert(msg); // anything you come up with
  }
});
~~~

That's it. After this every Netzke component will get this custom implementation of the `netzkeFeedback` method.

In the following tutorial I'll show you how exactly the same technique can be used to change the way Netzke handles session expiration (currently it would simply hint you in the browser console that the `onNetzkeSessionExpired` method should be overridden). Stay tuned.
