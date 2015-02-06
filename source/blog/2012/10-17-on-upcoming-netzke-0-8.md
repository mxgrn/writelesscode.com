---
title:      "On upcoming Netzke 0-8"
created_at: 2012-10-17
kind: article
tags: [netzke, 0-8, speaking]
excerpt: Netzke is moving towards a significant update, and here's a little high-level write-up on what I'm planning for the release.
---
Netzke is moving towards a significant update, and here's a little high-level write-up on what I'm planning for the release. This post will be followed by a series of more detailed post covering different topics in more details.

## On backward (in)compatibility
Version 0.8 will be a major, backward-incompatible update of Netzke gems. I decided to give up on backward compatibility due to that it's too difficult to make it right. Honestly, I never quite managed, and I don't want to try hard any longer. Instead, I will focus on making the code as clean, intuitive and maintainable as possible. The changelogs, as always, will be helpful during upgrades.

## Less magic
Last year I made a mistake taking the direction of introducing a lot of (kind of) nicely looking, but obscure and inflexible DSL methods that I used extensively while creating [Yanit](http://yanit.heroku.com), a demo app for a conference. In the end I realized that if there's a slightly more verbose - but less magic - way of achieving the same goal, it'll be more appreciated by developers, as there's much less to remember (and debug).

I also decided to unify the way how one defines child components, actions, endpoints. A separate blogpost on this will follow soon.

The code in both Core and Basepack has been profoundly re-written with the goal to simplify it, making it now much easier to understand, maintain, and use.

## Documentation
I will introduce more structure to the documentation:

* The project Readme on [GitHub](https://github.com/nomadcoder) will always be a good place to begin with.
* The detailed class documentation will be on [api.netzke.org](http://api.netzke.org) - I'm taking a significant effort to make it much more complete and consistent.
* The "official" tutorials will only be posted on this blog. I will [tag](/tags/) the posts with the actual Netzke version, so it'll be easier to find back the tutorials for older versions. I advise you to [subscribe](http://feeds.feedburner.com/writelesscodeblog) with your feed reader in order to stay up-to-date.
* The (improved) guides will still be located on [Netzke wiki](https://github.com/netzke/netzke/wiki).
* The wiki will also keep a list of community-contributed tutorials.
* Last but not least, some information about the releases and work in progress will be shared on [Twitter](https://twitter.com/netzke)

## Release date
I'll announce switching the master branch to 0-8 on the [Google Groups](http://groups.google.com/group/netzke/) and on [Twitter](https://twitter.com/netzke) within a couple of weeks. The gems will be released before [RubyConf Taiwan](http://rubyconf.tw/2012/) in December where I will be presenting Netzke (come say hi!)

That being said, the corresponding 0-8 branches of [Core](https://github.com/netzke/netzke-core), [Basepack](https://github.com/nomadcoder/netzke-basepack), [Demo](https://github.com/nomadcoder/netzke-demo), and [Yanit](https://github.com/nomadcoder/yanit) are pretty much usable already, so, I encourage you to play with them. Don't forget to check out the changelogs where applicable.
