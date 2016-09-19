---
layout: post
title: Improving Selenium IDE
author: Dave Hunt
date: 2010-07-07 12:57:12 +0000
tags: [selenium, selenium ide]
---
I've been working on a new formatter plugin for Selenium IDE, and along the way
I discovered some quirks (not necessarily bugs) in the code. After a few
discussions with some of the Selenium community, I decided to get stuck in and
see if I couldn't make some improvements. In the interest of sharing my
experiences, here is what I did.<!--more-->

# Format header/footer caching

For some reason (still not clear to me) the Selenium IDE formatter headers &
footers were cached in two places, the TestCase itself, and also within the
Format. I was finding that when switching between my new format and the default
HTML, whichever I used first was persisting as the header/footer for the other.
This could be fixed by using the Formatter cached header/footer but as I
couldn't see any advantage of caching this content, it seems like an unnecessary
and overcomplicated solution. So I
[removed the caching entirely](http://code.google.com/p/selenium/source/detail?r=9275).

# Updating variables in format source

The other main issue I had was when updating the base URL in Selenium IDE the
format source is not updated. By removing the caching I'd partially solved this,
but it still required the user to switch to another format and back again. Also,
changes to variables from the format options pane also need to update the format
source. I was able to find the appropriate place to
[initiate a refresh of the format source](http://code.google.com/p/selenium/source/detail?r=9276).

# Setting the initial base URL

Finally, I had an issue where the initial base URL in the format source was not
correct. When I tracked down the right place to put this minor fix I found the
line I needed was already there but commented out! After a quick check, it
appears that the change may have been a mistake, so I
[brought it back in](http://code.google.com/p/selenium/source/detail?r=9277) and
ran a few successful tests.

Obviously these changes are fairly significant, so the next release of Selenium
IDE (1.0.8) will need to be well tested. At this time there isn't a build
available with these changes, but if you are confident with running a
potentially unstable version you can
[check out from SVN](http://code.google.com/p/selenium/source/checkout) and
[build Selenium IDE yourself](http://code.google.com/p/selenium/wiki/BuildingSeIDE).

Please get in touch if you have any feedback. You can raise any issues you find
in the [official issue tracker](http://code.google.com/p/selenium/issues/list).
Please label them as 'ide'.

Stay tuned for more details on my new formatter - I have a few more issues to
work through before I'm ready to release it, and some of these may even involve
further improvements to Selenium IDE.
