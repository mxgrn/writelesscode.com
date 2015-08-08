---
title: Starting with Netzke 1.0 development
date: 2015-08-08 09:44 UTC
tags: [netzke]
excerpt: Thanks to the sponsor, I'll be able to spend a full month working on 1.0 release.
---

Guys from [PennyMac](http://www.pennymacusa.com) were extremely generous to offer me an opportunity to do a full month of paid Netzke development so that I could release version 1.0. For the coming 3 months, however, I'll be near-fulltime busy with other client work (React + Rails all the way), so, I plan to get full-steam on Netzke in November, with the deadline set to December 1st. However, this weekend the master on GitHub will already start pointing to 1.0.0.0.alpha, and I'll slowly start some development in my spare time (here's a very rough [list](https://gist.github.com/mxgrn/b5b4a6cb92c1b7d0c958) of what has to be done). Why the 4-digit version, suddenly? There's a good reason for that.

### Note on versioning

I have recently come up with an idea of using a 4-digit versioning schema for each of the [3](https://github.com/netzke/netzke-core) [Netzke](https://github.com/netzke/netzke-basepack) [gems](https://github.com/netzke/netzke-testing) (not including the [meta gem](https://github.com/netzke/netzke)):

* 1st digit will increase with each Ext JS major release (as soon as we start supporting it)
* 2nd digit will increase with each Rails major release (as soon as we start supporting it)
* 3rd digit will increase with each backward-incompatible changes in the Netzke API, as well as minor releases of Ext JS and/or Rails.
* 4th digit is for both bug fixes *and* backward-compatible new features

For example:

* 1.0.0.0 - Ext JS 5, Rails 4
* 1.0.1.0 - Ext JS 5, Rails 4, backward incompatible Netzke API
* 1.1.0.0 - Ext JS 5, Rails 5

* 2.0.0.0 - Ext JS 6, Rails 4 (or whatever newest Rails we need to support)
* 2.1.0.0 - Ext JS 6, Rails 5
* 2.2.0.1 - Ext JS 6, Rails 6, bugfixes

Changes in the first 2 digits will also mean that no backward compatibility of Netzke API is guaranteed, but the details on that, as before, will be provided in the CHANGELOG.

I'll start with supporting Ext JS 5 (and probably Rails 4), as moving to Ext JS 6 is not a priority for PennyMac (nor for my current in-house project, for that matter). You should [contact me](https://twitter.com/mxgrn), however, if your Netzke project needs to support Ext JS 6.

With that said, the [meta gem](https://github.com/netzke/netzke) will cut off the 4th digit and still use the 3-digit scheme, and will list its Netzke dependencies using the [pessimistic operator](https://robots.thoughtbot.com/rubys-pessimistic-operator) in its gemspec, for example:

    # in netzke.gemspec
    spec.version = "1.2.3"
    spec.add_dependency "netzke-core", "~> 1.2.3.0"
    spec.add_dependency "netzke-basepack", "~> 1.2.3.0"
    spec.add_dependency "netzke-testing", "~> 1.2.3.0"

This will guarantee that specifying the exact netzke gem version in your Gemfile, `bundle update` will still fetch all (hopefully) backward compatible updates:

    gem 'netzke', '1.2.3'

Should something go wrong with the last-digit release of Netzke, you always have the option of getting more specific:

    gem 'netzke-core', '1.2.3.14'
    gem 'netzke-basepack', '1.2.3.53'
    gem 'netzke-testing', '1.2.3.7'

Note, that the 4th digit won't necessarily be synced between Netzke gems.

### A word on blogging

I plan to [blog](http://writelesscode.com/) and [tweet](https://twitter.com/netzke) a bit more regularly on the new stuff coming in Netzke, as the 1.0 development progresses. I'll just try to keep the posts more brief and focused, so it doesn't feel like a huge undertaking each time any more.

Please, let me know in the comments what you think (and did I mention the [1.0 roadmap](https://gist.github.com/mxgrn/b5b4a6cb92c1b7d0c958) yet?)
