---
title:      "ExtJS/Rails CRUD Application in 7 Minutes"
created_at: 2010-06-14
kind: article
tags: [tutorials, netzke, 0-7]
excerpt: Build a full-featured Netzke task manager in no time.
---
<img align="right" class="frame-right" width="170" height="170" src="http://writelesscode.com/images/2010-06-14-extjswithrails.png" alt="ExtJS and Rails With"/>
***Update:** a [revised version](/blog/2012/10/20/extjs-rails-crud-application-in-7-minutes/) of this tutorial featuring [Netzke v0.8](/tags/0-8/) is available.*

This post will lead you through simple steps of creating a task manager web application with Ext JS, Ruby on Rails, and [Netzke](http://netzke.org) It will take you approximately 7 minutes to build, and if you're beforehand curious whether it's worth following, go straight to the last section but one (by far the biggest in this tutorial), where I discuss the results. The goal is to create a web application that will allow you to add, edit and remove TODO tasks, as well as mark them done. In addition to that, you'll be able to sort and search the tasks, edit several tasks at once, and more. You may start you stopwatch now, and let's get to the job.

> Writing this tutorial, I was using: Rails 3.2.6, netzke-core 0.7.6, netzke-basepack 0.7.6, Ruby 1.9.3, Ext JS 4.0.7.

## Setting things up

Create a new rails application:

<% highlight :text do %>
> rails new netzke_task_manager && cd netzke_task_manager
~~~

Add the Netzke gems to your Gemfile:

~~~ruby
gem 'netzke-core', '0.7.6'
gem 'netzke-basepack', '0.7.6'
~~~

Install the gems:

<% highlight :text do %>
> bundle install
~~~

Link the Ext library and, optionally, the [FamFamFam silk icons](http://www.famfamfam.com/lab/icons/silk/), for example (most probably no copy-pasting here!):

<% highlight :text do %>
> ln -s ~/code/extjs/ext-4.0.7 public/extjs
> mkdir public/images
> ln -s ~/assets/famfamfam-silk public/images/icons
~~~

Declare Netzke routes and uncomment the root map in routes.rb:

~~~ruby
NetzkeTaskManager::Application.routes.draw do
  netzke
  root :to => "welcome#index"
  # ...
end
~~~

Generate the welcome controller:

<% highlight :text do %>
> rails g controller welcome index
~~~

Don't forget to remove <tt>public/index.html</tt>.

In the application layout replace the default JavaScript and stylesheets inclusion with the <tt>netzke_init</tt> helper, so that the result looks like this:

~~~erb
<!DOCTYPE html>
<html>
<head>
  <title>Netzke Task Manager</title>
  <%= netzke_init %>
  <%= csrf_meta_tag %>
</head>
<body>
<%= yield %>
</body>
</html>
~~~

> Note that <tt>netzke_init</tt> is all what's needed to include Ext and Netzke JavaScript and stylesheets.

3 minutes passed, we're ready to get to the fun part.

## Creating the model

Let's create the Task model that will have a name, priority, notes, due date, and the "done" flag:

<% highlight :text do %>
> rails g model Task done:boolean name:string notes:text priority:integer due:date
~~~

Modify the migrations file (<tt>db/migrate/xxx_create_tasks.rb</tt>) slightly to have the "done" flag set to <tt>false</tt> by default:

~~~ruby
  t.boolean :done, :default => false
~~~

Run the migrations:

<% highlight :text do %>
> rake db:migrate
~~~

We want our task to always have at least the name set, so, let's add the proper validations. And set the default scope to only give us incomplete tasks:

~~~ruby
class Task < ActiveRecord::Base
  validates_presence_of :name
  default_scope :conditions => {:done => false}
end
~~~

## Embedding Netzke grid panel

We don't have much to do to see an Ext grid as an interface to our model. Simply declare Netzke GridPanel in <tt>app/views/welcome/index.html.erb</tt>:

~~~erb
<%= netzke :tasks, :class_name => "Netzke::Basepack::GridPanel", :model => "Task", :height => 400 %>
~~~

Start the server:

<% highlight :text do %>
> rails s
~~~

... and see how it looks on [http://localhost:3000/](http://localhost:3000/):

![Netzke Task Manager](/images/2010-06-14.jpg)

It's fully functional and nice-looking already. In a moment I'll provide you with an impressive list of all the things you can do with it, but first let's tweak it a bit so that it looks even nicer - we still have plenty of time for that.

With <tt>Netzke::Basepack::GridPanel</tt> you can easily customize the columns (see a [comprehensive tutorial](http://demo.netzke.org/grid_panel) about it). Let's do 2 simple things here: 1) provide the list of the columns that we want to see, excluding the <tt>created_at</tt> and <tt>updated_at</tt> columns that Rails adds by default, and 2) change the title of the "due" column to "Due on".

~~~erb
<%= netzke :tasks,
  :class_name => "Netzke::Basepack::GridPanel",
  :model => "Task",
  :height => 400,
  :columns => [:done, :name, :notes, :priority, {:name => :due, :header => "Due on"}]
%>
~~~

Perfect. Let's use our last 2 minutes to do the final - purely visual - touch. Let's display our grid in the middle of the page, under a title, without that thick blue header, and with a nice border around. And also let's adjust some columns' default width and make them automatically occupy the whole available width of the grid.

To put the grid in the middle of the page, let's quickly add some inline styles into the application layout (after the <tt>netzke_init</tt> helper):

~~~erb
<style type="text/css" media="screen">
  h1 { text-align: center; margin: 10px;}
  .netzke-component { width: 700px; margin: auto; }
</style>
~~~

To add a title, enable the border and disable the grid's header, update the view:

~~~erb
<h1>Incomplete tasks</h1>

<%= netzke :tasks,
  :class_name => "Netzke::Basepack::GridPanel",
  :model => "Task",
  :height => 400,
  :columns => [:id, :done, :name,
    {:name => :notes, :width => 200},
    {:name => :priority, :width => 50},
    {:name => :due, :header => "Due on"}
  ],
  # Standard Ext.grid.EditorGridPanel configuration options:
  :border => true,
  :header => false,
  :view_config => {
    :force_fit => true # force the columns to occupy all the available width
  }
%>
~~~

Well, that's it! Stop your stopwatch, and let's discuss in details what we've got:

![(Netzke Task Manager Complete)](/images/2010-06-14-2.jpg)

## Discussing the results

Because <tt>Netzke::Basepack::GridPanel</tt> is a very powerful Netzke component, our application gets a lot of features for free.

### Multi-line CRUD operations

Adding, updating and deleting records can easily be done in a multiline way:

![(grid panel multiline CRUD)](/images/2010-06-14-multiline.jpg)

### Pagination

Even if you data table contains tens of thousands of records, it's no problem for a Netzke grid panel, thanks to the built-in pagination.

### Context menu

Some of the button actions on the bottom of the grid are duplicated in the grid's context menu:

![(grid panel context menu)](/images/2010-06-14-context-menu.jpg)

### Automatic detection of attribute types

In our application we're using an integer, boolean, string, text, and date fields in the Task model - and each of them are getting its own column type (you'll not be able to enter letters in the priority field).

### Support for Rails validations

Rails validations are respected (and they play nicely with multiline editing!):

![(grid panel validations)](/images/2010-06-14-validations.jpg)

### Server-side sorting

Click the header of the column to enable server-side sorting:

![(grid panel sorting)](/images/2010-06-14-sorting.jpg)

### Server-side filtering

Smart filters are enabled by default by each column, corresponding to the data type.

For due-date:

![(grid panel filtering by date)](/images/2010-06-14-filtering-1.jpg)

For priority:

![(grid panel filtering by priority)](/images/2010-06-14-filtering-2.jpg)

### Adding/(multi-)editing records in a form

Sometimes adding/editing a record is much handier from within a form. Netzke gives you that, too. And even multi-record editing is supported - just select multiple rows and press "Edit in form".

![(grid panel editing in form)](/images/2010-06-14-editing-in-form.jpg)

### Advanced search, with presets management

> Note: presets management was temporally left out during the move to Rails 3, but will soon be back.

![(grid panel advanced search)](/images/2010-06-14-advanced-search.jpg)

### And more

While not covered in this tutorial, Netzke grid panel also supports:

* one-to-many ("belongs_to") relationships (see a link for a demo below)

## Conclusion

You've learnt about a small part of what Netzke can provide, on an example of <tt>Netzke::Basepack::GridPanel</tt> - a very powerful, customizable and extendible component, that you can use in your RIA. You can see more examples of using GridPanel and other Netzke components on [live demo](http://demo.netzke.org) In fact, Netzke was conceived as a framework that let's you create your own powerful components - either from scratch, or using existing components as building parts.

Follow me on [twitter](http://twitter.com/mxgrn) to stay updated about Netzke, and don't forget to bookmark the official Netzke [project website](http://netzke.org). Share your Netzke experience on the [Google Groups](http://groups.google.com/group/netzke), and - last but not least! - please, *spread the word*: Netzke is a many-sided project where a larger community will mean faster development. Thanks!

_Credits for the ExtJsWithRails logo go to "www.extjswithrails.com":http://www.extjswithrails.com_
