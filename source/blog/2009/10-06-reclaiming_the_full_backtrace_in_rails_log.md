---
title:      Reclaiming the Full Backtrace in Rails' Log
created_at: 2009-10-06
kind: article
outdated: true
tags: [rails]
---
While debugging AJAX-reach Rails applications (which is the case when you use ExtJS) with Rails >= 2.3.0, you wish you could get the full backtrace of an exception written to <tt>development.log</tt>, rather than sent to the browser (where you don't see it anyway unless you dig into the Firebug log). It cost me some futile searching on the Internet in pursue to figure out how to reclaim the full backtrace in the logs, while the solution appears to be extremely simple - you just have to know where to look.

Can't say why, but I never paid attention to the <tt>backtrace_silencers.rb</tt> file that can be found in <tt>config/initializers</tt> (if you don't have this file, you can create it, of course). In this file you can simply uncomment the following line in order to make Rails write full backtrace of the exceptions into the log:

<% highlight :ruby do %>
# Rails.backtrace_cleaner.remove_silencers!
<% end %>

Besides that, I added the following "silencer" in order to hide hardly informative passenger-related lines:

<% highlight :ruby do %>
Rails.backtrace_cleaner.add_silencer { |line| line =~ /passenger/ }
<% end %>

Based on this example, you can specify (as a regular expression) what is that that you never want to see in the backtrace.

Hope, this saves time to someone.