---
title:      "Testing with CoffeeScript and Mocha"
created_at: 2013-02-25
kind: article
tags: [netzke, testing, 0-8]
excerpt: Specs for Netzke Core have been rewritten to make use of Mocha and CoffeeScript. Here I'll briefly show what I did, so you could consider this setup for testing your own Ext JS + Rails applications.
---
Integration testing of Netzke Core requires running a lot of JavaScript, which, before the rewrite, resulted in numerous custom Cucumber steps that served as wrappers around that JavaScript code. The tests used to read [pretty nicely](https://github.com/netzke/netzke-core/blob/v0.8.1/test/core_test_app/features/component_loader.feature), but the code overhead was too high, and performance very low, as for each spec a page containing the tested component would get reloaded. So, I decided to have a look at JavaScript testing frameworks, and after trying out [Jasmine](http://pivotal.github.com/jasmine/) first, decided that [Mocha](http://visionmedia.github.com/mocha/) would serve me better (thanks @ptico for [pointing me](https://twitter.com/ptico/status/292310840013619202) in that direction). After enabling CoffeeScript for writing actual specs, and creating a few custom Ext JS-related helpers, I ended up with specs that read almost as good as Cucumber features,  at the same time written in a *real* programming language, not Gherkin:

<% highlight :coffeescript do %>
describe "Actions component", ->
  it "should handle clicking a button", ->
    click button "Simple action"
    expectToSee header "Simple action triggered"

  it "should show certain buttons disabled", ->
    expectDisabled button "Disabled action"

  it "should not show buttons for excluded actions", ->
    expectToNotSee button "Excluded action"
<% end %>

And for asynchronous scenarios:

<% highlight :coffeescript do %>
describe "Plugins component", ->
  it "should be able to call its server part as defined in PluginWithEndpoints", (done) ->
    click tool 'gear'
    wait ->
      expectToSee header "Response from server side of PluginWithEndpoints"
      done()
<% end %>

You can find the rest of the Mocha features in [spec/mocha](https://github.com/netzke/netzke-core/tree/master/spec/mocha). Every file from that folder, corresponding to an equally named testing component, is being automatically picked up by the setup and turned into a separate RSpec spec. This way there's generally one Mocha feature for each component being tested. The specs inside that feature are being run all at once, without the need to reload the page. As a result, it's much faster now (also due to the fact that the assertions are now being executed directly in the browser).

Individual features can be run manually in any browser (e.g. in order to analyze the failures) by appending "?spec={feature}" to the URL of a tested component, for example: `http://core-test-app.dev/components/Endpoints?spec=endpoints`. This will run `spec/mocha/endpoints.js.coffee` spec on the `test/core_test_app/components/endpoints.rb` component:

![Mocha spec](/images/2013-02-25.mocha-spec.png)

I also want to add that most of the testing components have been re-worked and better named in order to give the developer a clear message of what exactly is being tested. Hopefully this will make understanding Netzke code an easier task, too.
