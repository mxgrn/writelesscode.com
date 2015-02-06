---
title:      "Ext JS and Ruby on Rails CRUD Application in 7 Minutes"
created_at: 2012-10-20
kind: article
tags: [tutorials, netzke, 0-8]
excerpt: Build a Ruby on Rails and Sencha Ext JS task manager web application in no time.
---
*This is a revision of the [older tutorial](/blog/2010/06/14/extjs-rails-crud-application-in-7-minutes/), to which you can refer in case you're using Netzke 0.7.x.*

This post will lead you through simple steps of creating a task manager web application with Ext JS, Ruby on Rails, and [Netzke](http://netzke.org). It will take you approximately 7 minutes to build, and if you're beforehand curious whether it's worth following, go straight to the last section but one (by far the biggest in this tutorial), where I'm discussing the results. The goal is to create a web application that will allow you to add, edit and remove TODO tasks, as well as mark them done. In addition to that, you'll be able to sort, filter and search the tasks, edit/delete several tasks at once, and more. You may start you stopwatch now, and let's get to the job.

> While working on this tutorial I was using Rails 3.2.8, netzke-core v0.8.0, netzke-basepack v0.8.0, Ruby 1.9.3, Ext JS 4.1.1a.

## Setting things up

Create a new Rails application:

    $ rails new netzke_task_manager && cd netzke_task_manager

Add Netzke gems to your Gemfile:

~~~ruby
gem 'netzke-core', '~>0.8.0'
gem 'netzke-basepack', '~>0.8.0'
~~~

Install the gems:

    $ bundle install

Link the [Ext JS files](http://www.sencha.com/products/extjs/download/ext-js-4.1.1) and, optionally, the [FamFamFam silk icons](http://www.famfamfam.com/lab/icons/silk/), into `public/extjs` and `public/images/icons` respectively. For example:

    $ echo Most probably no copy-pasting here!
    $ ln -s ~/code/extjs/ext-4.1.1 public/extjs
    $ mkdir public/images
    $ ln -s ~/assets/famfamfam-silk public/images/icons

Declare Netzke routes and uncomment the root map in `config/routes.rb`:

~~~ruby
NetzkeTaskManager::Application.routes.draw do
  netzke
  root to: "welcome#index"
end
~~~

Generate the welcome controller:

    $ rails g controller welcome index

Don't forget to remove `public/index.html`.

In `app/views/layouts/application.html.erb` replace the default JavaScript and stylesheets inclusion with the `load_netzke` helper, so that the result looks like this:

~~~erb
<!DOCTYPE html>
<html>
<head>
  <title>Netzke Task Manager</title>
  <%%= load_netzke %>
  <%%= csrf_meta_tag %>
</head>
<body>
<%%= yield %>
</body>
</html>
~~~

> Note that `load_netzke` is all what's needed to include Ext JS and Netzke scripts and stylesheets.

3 minutes passed, we're ready to get to the fun part!

## Creating the model

Let's create the Task model that will have a name, priority, notes, due date, and the "done" flag:

    $ rails g model Task done:boolean name notes:text priority:integer due:date

Run the migrations:

    $ rake db:migrate

We want our task to always have at least the name set, so, let's add proper validations (in `app/models/task.rb`):

~~~ruby
class Task < ActiveRecord::Base
  attr_accessible :done, :due, :name, :notes, :priority
  validates_presence_of :name
end
~~~

## Creating the Tasks grid component
Let's create our first Netzke component based on a pre-built full-featured `Netzke::Basepack::Grid`. First let's create the `app/components` folder:

    $ mkdir app/components

In that folder let's create a file called tasks.rb with the following content:

~~~ruby
class Tasks < Netzke::Basepack::Grid
  def configure(c)
    super
    c.model = "Task"
  end
end
~~~

Our Netzke component is a Ruby class that inherits from `Grid` and is configured to use the previously created `Task` model. Now we need to embed it into our view. In `app/views/welcome/index.html.erb`, replace the default content with the following line:

~~~erb
<%%= netzke :tasks, height: 400 %>
~~~

Start the server:

    $ rails s

... and see how it looks on [http://localhost:3000/](http://localhost:3000/):

![Netzke Task Manager](/images/2010-06-14.png)

It's fully functional and nice-looking already. In a moment I'll provide you with an impressive list of all the things you can do with it, but first let's tweak it a bit so that it looks even nicer - we still have plenty of time for that.

`Netzke::Basepack::Grid` is very customizable. Let's do 4 simple things here:

1. provide the list of the columns that we want to see (excluding the `created_at` and `updated_at`)
2. change the title of the "due" column to "Due on"
3. make the "notes" column fill the available width, by using the `flex` config option understood by [Ext.grid.column.Column](http://extjs.dev/docs/index.html#!/api/Ext.grid.column.Column)
4. use `Netzke::Basepack::Grid`'s `scope` config option to filter out those records that have their `done` flag set

Our resulting component will look like this:

~~~ruby
class Tasks < Netzke::Basepack::Grid
  def configure(c)
    super
    c.model = "Task"
    c.columns = [
      :done,
      :name,
      {name: :notes, flex: 1},
      :priority,
      {name: :due, header: "Due on"}
    ]
    c.scope = {done: [nil, false]}
  end
end
~~~

Perfect. Let's use our last 2 minutes to do the final - purely visual - touch. To put the grid in the middle of the page, let's quickly add some inline styles into the application layout (right after the `load_netzke` helper):

~~~erb
<style type="text/css" media="screen">
  h1 { text-align: center; margin: 10px;}
  .netzke-component { width: 800px; margin: auto; }
</style>
~~~

Add the h1 header to the index page:

~~~erb
<h1>Incomplete tasks</h1>
<%%= netzke :tasks, height: 400 %>
~~~

Well, that's it! Stop your stopwatch, and let's discuss in details what we've got:

![(Netzke Task Manager Complete)](/images/2010-06-14-2.png)

## Discussing the results

Because `Netzke::Basepack::Grid` is a very powerful Netzke component, our application gets a lot of features for free. Let's go through them.

### Automatic detection of attribute types

In our application we're using an integer, boolean, string, text, and date fields in the Task model - and each of them are getting its own column type (for example, you'll not be able to enter letters in the priority field, there's a date picker for the date field, etc).

### Pagination

Even if you data table contains tens of thousands of records, it's no problem for a Netzke grid panel, thanks to the built-in pagination.

### Multi-line CRUD operations

Adding, updating and deleting records can easily be done in a multi-line way:

![(grid panel multiline CRUD)](/images/2010-06-14-multiline.png)

### Context menu

Some of the button actions on the bottom of the grid are duplicated in the grid's context menu:

![(grid panel context menu)](/images/2010-06-14-context-menu.png)

### Support for Rails validations

Rails validations are respected (and they play nicely with multi-line editing!):

![(grid panel validations)](/images/2010-06-14-validations.png)

### Server-side sorting

Click the header of a column to enable server-side sorting:

![(grid panel sorting)](/images/2010-06-14-sorting.png)

### Server-side filtering

Smart filters are enabled by default by each column, according to the column's field type.

For due-date:

![(grid panel filtering by date)](/images/2010-06-14-filtering-1.png)

For priority:

![(grid panel filtering by priority)](/images/2010-06-14-filtering-2.png)

### Adding/(multi-)editing records in a form

Sometimes adding/editing a record is much handier from within a form. Netzke gives you that, too. And even multi-record editing is supported - just select multiple rows and press "Edit in form".

![(grid panel editing in form)](/images/2010-06-14-editing-in-form.png)

### Advanced search, with presets management

Press the Search button to build complex search queries.

![(Grid panel advanced search)](/images/2012-10-20-advanced-search.jpg)

## Just a tip of an iceberg

What you have learned is only a tiny fraction of what Netzke is capable of. In fact, you only got a bit acquainted with how to use a pre-built component, which was in its turn created by using the extremely powerful [Netzke Core](https://github.com/netzke/netzke-core). Some things that it allows you is combining components into other (composite) components, loading components dynamically, implement complex client-server interactions, enhance components behaviour in an object-oriented way (this one we did address by inheriting `Tasks` from `Netzke::Basepack::Grid`) - and much more, which makes Netzke, leveraging the power of Ext JS and Rails, an ideal framework for building extremely complex "rich Internet applications" (RIA).

On the continuation of this post, I'll show you how easy it is to put 3 instances of our `Tasks` component on a tab panel as separate tabs, each showing a different set of records: completed tasks, not completed tasks, and all tasks. Check out the [next part of the tutorial](/blog/2012/12/17/nesting-netzke-components/).
