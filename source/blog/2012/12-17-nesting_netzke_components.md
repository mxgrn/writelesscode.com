---
title:      "Nesting Netzke components"
created_at: 2012-12-16
kind: article
tags: [netzke, tutorials, 0-8]
excerpt: On the continuation of the popular tutorial teaching you how to set up a <a href="/blog/2012/10/20/extjs-rails-crud-application-in-7-minutes/">CRUD web app in 7 minutes</a>, I'll show you how easy it is to nest existing Netzke components in a tab panel.
---

As I mentioned in the end of the [previous tutorial](/blog/2012/10/20/extjs-rails-crud-application-in-7-minutes/), Netzke lets you do much more than just quickly create a grid that seamlessly works with an ActiveRecord model without using any code generators. In fact, Netzke is [best suited](http://netzke-rubyconf-taiwan-2012.herokuapp.com) for making complex one-page web applications that make use of hundreds of data models and implement sophisticated workflows. One of the many techniques that Netzke as a modular GUI framework provides to you, is component nesting. Here I'll show you how easy it is to nest 3 differently configured instances of the Tasks component created before in a tab panel.

## Creating tab panel

Assuming you already have the [first part](/blog/2012/10/20/extjs-rails-crud-application-in-7-minutes/) of the tutorial up-and-running, let's start by creating a new component called `TaskTabPanel`, and indicate that its [client class](http://rdoc.info/github/netzke/netzke-core#Client_class) should inherit from [Ext.tab.Panel](http://docs.sencha.com/ext-js/4-1/#!/api/Ext.tab.Panel):

~~~ruby
class TaskTabPanel < Netzke::Base
  js_configure do |c|
    c.extend = "Ext.tab.Panel"
  end
end
~~~

Update `views/welcome/index.html.erb` to show our newly created component:

~~~erb
<h1>Tasks</h1>
<%%= netzke :task_tab_panel, height: 400 %>
~~~

If we reload our application now, it won't be much, as our tab panel has no tabs yet. We want to see 3 tabs containing our previously created Tasks component, each showing a different subset of tasks.

## Nesting Tasks component

We need 2 simple steps to nest a few instances of an existing Netzke component in our tab panel. First, let's declare them as child components inside `TaskTabPanel`:

~~~ruby
class TaskTabPanel < Netzke::Base
  js_configure do |c|
    c.extend = "Ext.tab.Panel"
  end

  component :incomplete_tasks do |c|
    c.klass = Tasks
  end

  component :completed_tasks do |c|
    c.klass = Tasks
  end

  component :all_tasks do |c|
    c.klass = Tasks
  end
end
~~~

Now, with child components declared, Netzke gives us 2 choices on how to show them to the user. We could nest them statically in the current component, or dynamically load them on the user request using `netzkeLoadComponent` client-class method. Let's do the simplest thing now, and add all 3 of them statically as tabs. For this we use Ext JS's usual `items` property:

~~~ruby
class TaskTabPanel < Netzke::Base
  js_configure do |c|
    c.extend = "Ext.tab.Panel"
  end

  component :incomplete_tasks do |c|
    c.klass = Tasks
  end

  component :completed_tasks do |c|
    c.klass = Tasks
  end

  component :all_tasks do |c|
    c.klass = Tasks
  end

  def configure(c)
    super
    c.items = [:incomplete_tasks, :completed_tasks, :all_tasks]
  end
end
~~~

Note, that we didn't have to provide the detailed configuration for the tabs, and just used Symbols referring the child components - Netzke does the expansion of those behind the scenes.

Now we have 3 tabs, each with an instance of `Tasks` component, but they are all named equally and show the same list of incomplete tasks:

![3 Tasks components as tabs](/images/2012-12-17-3-tabs.png)

## Configuring nested components

One way to change this would be by extending our `Tasks` component 3 times and configure each of them separately, for example, for completed tasks:

~~~ruby
class CompletedTasks < Tasks
  def configure(c)
    super
    c.title = "Completed"
    c.scope = {done: true}
  end
end
~~~

Then we could use it without any further configuration in `TaskTabPanel` by specifying `c.klass = CompletedTasks` inside the `component` block. However, in this case it would be an overkill, and we can simply configure the 3 nested grids directly in `TaskTabPanel`:

~~~ruby
component :incomplete_tasks do |c|
  c.klass = Tasks
  c.title = "Incomplete"
  c.scope = {done: [nil, false]}
end

component :completed_tasks do |c|
  c.klass = Tasks
  c.title = "Completed"
  c.scope = {done: true}
end

component :all_tasks do |c|
  c.klass = Tasks
  c.title = "All"
  c.scope = nil
end
~~~

After reloading the page, we see that the titles indeed have been changed, but our scope configuration didn't take any effect:

![Title updated](/images/2012-12-17-fixed-titles.png)

The problem is in our previously created `Tasks` component, namely in the way we specify the scope there:

~~~ruby
class Tasks < Netzke::Basepack::Grid
  def configure(c)
    super
    # ...
    c.scope = {done: [nil, false]}
  end
end
~~~

As you can see, the scope is set *after* calling `super`, where the configuration from the `component` block is being applied. By moving it *before* `super` we'll ensure that it will get overridden from `TaskTabPanel`, thus making our `Tasks` component more configurable:

~~~ruby
class Tasks < Netzke::Basepack::Grid
  def configure(c)
    c.scope = {done: [nil, false]}

    super
    # ...
  end
end
~~~

> Naturally, we could also remove it completely if we don't need any default scopes in `Tasks`

Now we have it fixed:

![Final result](/images/2012-12-17-final.png)

## Discussing the results

You have learnt how to build a simple compound component, and how to make a nested component configurable, which is an alternative to extending a component by sub-classing it. Both techniques have their values in different scenarios, so, it's up to the software designer which approach to choose.

## Bonus: load tabs dynamically with Basepack::TabPanel

In our current implementation all the 3 instances of `Tasks` are being instantiated and added to the tab panel at the page load. This can be a performance issue when you have many tabs each containing a complex Ext JS component as grid. An alternative approach would be to load the nested components dynamically at the moment the user activates a tab for the first time. A dedicated component in netzke-basepack can do just that. All we need to do is to inherit our TaskTabPanel from `Netzke::Basepack::TabPanel`, and remove the `js_configure` section:

~~~ruby
class TaskTabPanel < Netzke::Basepack::TabPanel
  component :incomplete_tasks do |c|
    c.klass = Tasks
    c.title = "Incomplete"
    c.scope = {done: [nil, false]}
  end

  component :completed_tasks do |c|
    c.klass = Tasks
    c.title = "Completed"
    c.scope = {done: true}
  end

  component :all_tasks do |c|
    c.klass = Tasks
    c.title = "All"
    c.scope = nil
  end

  def configure(c)
    super
    c.items = [:incomplete_tasks, :completed_tasks, :all_tasks]
  end
end
~~~

![Dynamic tabs loading](/images/2012-12-17-dynamic-tabs.png)
