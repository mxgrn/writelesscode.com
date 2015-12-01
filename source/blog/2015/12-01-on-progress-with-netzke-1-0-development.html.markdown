---
title: "On progress with Netzke 1.0 development"
date: 2015-12-01 00:00 UTC
tags: [netzke]
excerpt: A small report on the progress with Netzke 1.0 development.
---

Since August I've been working on Netzke 1.0 release. As I was progressing, I realized I was too optimistic about releasing today, so, this should happen some time later this month. Partially, the reason for the delay was that my internal list of TODOs was growing faster than items on [this gist](https://gist.github.com/mxgrn/b5b4a6cb92c1b7d0c958) were getting checked off - I wanted to use the chance to rethink many different places in the code that were bothering me for very long, e.g. a few days went to rethinking the API alone. Still, some of the bigger challenges are now behind, and today I want to share some new things with you.

The updated grid component, by default, now uses 'infinite scrolling' instead of pagination (which is still an option, of course). Today I launched the ["edge" version](http://edgedemo.netzke.org) of the official Netzke demo, in order to show off this new feature. [One of its grids](http://edgedemo.netzke.org/#clerks) got the seed volume increased to 20K records - this is for those of you who could question the performance (all credits go to Sencha). Another thing to note is that the amount of toolbar buttons is reduced. That's due to that adding/updating records now, by default, takes place via a form, while in-line editing has become [another option](http://edgedemo.netzke.org/#grid_with_inline_editing).

For (growing) detailed list of changes coming to Netzke 1.0, refer to the changelog of [Core](https://github.com/netzke/netzke-core/blob/master/CHANGELOG.md) and [Basepack](https://github.com/netzke/netzke-basepack/blob/master/CHANGELOG.md). Note, that the API is not yet fixed and will probably still be changed here and there before the release. And I'm aware of a few bugs and glitches - working on them, too.

Thanks again to [PennyMac](http://www.pennymacusa.com/) for generously sponsoring this work.
