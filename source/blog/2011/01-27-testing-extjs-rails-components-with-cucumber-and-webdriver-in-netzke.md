---
title:      Testing Ext JS/Rails components with Cucumber and WebDriver in Netzke
created_at: 2011-01-27
kind: article
tags: [netzke, testing]
excerpt: This brief blogpost will show you an example of a typical test scenario in Netzke Basepack, along with the code for a Cucumber step that communicates to the Ext JS component being tested.
---
<img align="right" class="frame-right" width="209" height="70" src="http://writelesscode.com/images/2011-01-27.jpg" alt="Netzke Basepack Cucumber tests"/>If you're maintaining an open source project, a peace of mind is impossible without a decent test coverage. I've been successfully testing Netzke using Cucumber and Selenium/WebDriver (with some complementary RSpec tests as well). In this brief blogpost I will show you an example of a typical test scenario in Netzke Basepack, along with the code for a Cucumber step that communicates to the Ext JS component being tested. At the end, you can enjoy a little video I recorded of running complete Cucumber test suite on Basepack - I just thought it looked fun!

Here's an example of a scenario that tests inline editing in `Basepack::GridPanel`:

    Scenario: Inline editing
      Given a book exists with title: "Magus", exemplars: 100
      When I go to the BookGrid test page
      And I edit row 1 of the grid with title: "Collector", exemplars: 200
      And I press "Apply"
      And I wait for the response from the server
      Then the grid should have 0 modified records
      And a book should exist with title: "Collector", exemplars: 200
      But a book should not exist with title: "Magus"

This scenario makes use of the BookGrid component [defined](https://github.com/netzke/netzke-basepack/blob/master/test/rails_app/app/components/book_grid.rb) in the test application bundled with Basepack.

And here's the implementation of the step that checks the amount of modified records in the grid:

    Then /^the grid should have (\d+) modified records$/ do |n|
      page.driver.browser.execute_script(<<-JS).should == n.to_i
        var components = [];
        for (var cmp in Netzke.page) { components.push(cmp); }
        var grid = Netzke.page[components[0]];
        return grid.getStore().getModifiedRecords().length;
      JS
    end

It's simply assuming that the first component on the page is the grid that we're testing. Other than that, both the scenario and the step implementation are rather straightforward, aren't they? As you may notice, I'm using the functionality of the Ext JS grid itself to do the necessary check - thus not messing with the generated DOM directly. I find this approach working pretty well.

I hope this blogpost will inspire you to write tests for your own components if aren't doing this already. You may browse the features and test applications for [netzke-core](https://github.com/netzke/netzke-core) and [netzke-basepack](https://github.com/nomadcoder/netzke-basepack) to see more examples of how Ext JS components can be tested with Cucumber. Below is a little video that demos the complete run of the Cucumber tests for Netzke Basepack (in its actual state).

Happy testing, and don't forget to follow me on Twitter [here](http://twitter.com/mxgrn)!

<iframe src="http://player.vimeo.com/video/19254788" width="400" height="300" frameborder="0"></iframe><p><a href="http://vimeo.com/19254788">Netzke Basepack tests</a> from <a href="http://vimeo.com/nomadcoder">Nomad Coder</a> on <a href="http://vimeo.com">Vimeo</a>.</p>
