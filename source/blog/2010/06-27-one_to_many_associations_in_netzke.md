---
title:      One-to-Many Associations in Netzke
created_at: 2010-06-27
kind: article
outdated: true
tags: [tutorials, netzke]
excerpt: Using one-to-many associations in Netzke grids and forms.
---
<img align="right" class="frame-right" width="320" height="280" src="http://writelesscode.com/images/2010-06-27.jpg" alt="One-to-many associations in Netzke"/>Let's say we have a grid listing clerks, and we want to be able to assign a boss to a clerk (yes, my "favorite setup":http://demo.netzke.org/grid_panel). A traditional way to do this is by using a combobox listing available bosses as options. Netzke does it for you automatically when it detects a foreign key between the model's attributes. However, sometimes we want to be in more control over how one-to-many associations are handled in Netzke grids and panels. This tutorial will show you how to configure the following:
* How each boss should be represented in the drop-down list. Will it be her first name or last name? Or maybe an arbitrary string that combines them both and eventually includes some extra info (such as the boss' salary)?
* Scoping out bosses in the drop-down list. We may not want to list all the bosses in the drop-down list. How could we limit those to a specific department, for instance?

> The rest of the tutorial is hosted on the Netzke wiki on GitHub. Please, help me keep it up-to-date by editing the wiki.

"Continue reading the tutorial":https://github.com/netzke/netzke/wiki/One-to-many-associations.
