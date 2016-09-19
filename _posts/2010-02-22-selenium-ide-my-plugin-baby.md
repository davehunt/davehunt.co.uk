---
layout: post
title: Selenium IDE - My plugin baby
author: Dave Hunt
date: 2010-02-22 09:28:53 +0000
tags: [selenium, selenium ide, seleniumexamples.com]
---
With [last month's release of Selenium IDE 1.0.4](http://saucelabs.com/blog/index.php/2010/01/selenium-ide-1-0-4-is-released/)
came initial support for plugins, and
[this month's release of 1.0.5](http://saucelabs.com/blog/index.php/2010/02/selenium-ide-1-0-5-is-released/)
continues to build on the new plugin API as well as fixing a few bugs.<!--more-->

This is quite possibly the most exciting development in Selenium IDE in a long
time, and the best thing is that it enables users to come up with their own
improvements - in much the same way as [Greasemonkey](http://www.greasespot.net/)
or Firefox add-ons themselves. The possibilities are endless.

Adam Goucher continues to post guidance on creating plugins
[on his blog](http://adam.goucher.ca/?s=The+Selenium-IDE+1.x+plugin+API) as the
API evolves. I have followed these myself to create two plugins:

# Flow control
This is actually just an old Selenium user extension that I've repackaged into
a plugin. Two main advantages of doing this are: an easy
install/disable/uninstall process, and no messing around with JavaScript files!
[[install]](https://addons.mozilla.org/en-US/firefox/downloads/file/81157/selenium_ide__flow_control-1.0.1-fx.xpi)
[[public repository]](http://github.com/davehunt/selenium-ide-flowcontrol)

# WebDriver Backed Selenium Formatters
This is a formatter plugin that provides the Selenium emulation from WebDriver,
giving quick access to WebDriver's advantages without changing from the existing
Selenium API.
[[install]](https://addons.mozilla.org/en-US/firefox/downloads/file/81150/selenium_ide__webdriver_backed_formatters-1.0.1-fx.xpi)
[[public repository]](http://github.com/davehunt/selenium-ide-webdriver-backed-formatters)

These plugins are available to download, and you can view the source/report
issues/provide feedback via the GitHub repositories.

Let me know if you're developing your own plugin, or have any good ideas. I'll
be happy to review, host, or link to your plugins - most of all we want to
encourage plugin development.

**Update:** Plugins now made available via addons.mozilla.org:

 * [Flow Control](https://addons.mozilla.org/en-US/firefox/addon/85794)
 * [WebDriver Backed Selenium Formatters](https://addons.mozilla.org/en-US/firefox/addon/85793)
