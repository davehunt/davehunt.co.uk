---
layout: post
title: Improving Selenium Grid
author: Dave Hunt
date: 2009-11-27 17:12:31 +0000
tags: [selenium, selenium grid, seleniumexamples.com]
---
Hopefully Selenium Grid 1.0.5 will soon be released, with the much anticipated
self-healing features that will save me so much time when RCs go AWOL. Looking
further ahead, I would like to see some minor improvements to the Selenium Grid
Console such as integrating
[this very handy Greasemonkey script for unregistering Remote Controls](http://userscripts.org/scripts/show/49061)
and sortable columns.<!--more--> Below is a quick mock-up of how the console could look
with these simple changes.

![Selenium Grid Improvements]({{ site.github.url }}/assets/selenium-grid-improved1.png)

The above changes I'm pretty confident I could make myself, but after trying to
get Selenium Grid with a resource handler for a couple of hours I decided
instead I'd mock it up and make a public plea for these changes!

While I'm at it, something I'm less certain I'd be able to do myself is add the
queued requests to the console with the ability to delete them. It's not
uncommon for someone to fire off a test suite and when the first few tests
appear to be failing, cancel the process. Unfortunately this means that several
RCs are unable to complete, and Grid still has pending requests. In the past
I've started X number of RCs in order to consume the remaining requests and then
kill them, however I usually just restart the Grid and all RCs. Below is an idea
of how this could look in the console.

![More Selenium Grid Improvements]({{ site.github.url }}/assets/selenium-grid-improved2.png)
