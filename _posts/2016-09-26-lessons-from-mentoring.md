---
layout: post
title: Lessons from mentoring
author: Dave Hunt
date: 2016-09-26 18:45:03 +0100
categories:
- mozilla
tags:
- mentoring
comments: true
---
Now that my first two mentoring experiences have come to an end, I wanted to
reflect a little on the experience. Before I do, I want to say that I'm
incredibly proud of what [Justin] and [Ana] were able to achieve this summer. I
feel lucky to have had the opportunity to mentor such talented, enthusiastic,
and motivated individuals. I hope they feel they've learned something of value
during their projects - I most certainly have!

Something to note, before I get into my learnings - and advice for anyone about
to embark on mentoring (or being mentored) - is that these were remote
mentorships. Although I was fortunate enough to spend a little in-person time
with Justin and Ana, for the majority of their projects we had an ocean
separating us. As a result some of what I'll talk about may be specific to the
nature of this challenge.

## Prepare a cohesive project

The two projects I mentored were very different. Ana's project was to enhance
a python testing plugin that generates HTML reports to work under the
restrictions enforced by a strict [Content Security Policy][CSP]. Justin's
project was to improve the automated testing for Firefox add-ons and associated
services such as [addons.mozilla.org][AMO].

As Ana's project was an enhancement to a [pytest plugin][pytest-html] that I
maintain, I already understood the problems that the project addressed, which
helped me to form a fairly cohesive roadmap. Initially, there were a few smaller
enhancements to get her familiar with the codebase and associated packages,
leading up to the main enhancement. This made it easier to gauge how well Ana
was progressing, and to decide when she was ready to move onto the main task.

For Justin's project, I was initially less familiar with the needs of the teams
involved. Although I had worked on the projects myself, it was much more
challenging to pull things together and see the bigger picture. I did at least
feel conscious of this throughout, and made an effort to present something
cohesive, but it didn't feel as natural to me.

I suspect this may be somewhat unique to my perspective as the mentor and having
confidence that I know what my intern will work on from one day to the next.
Nevertheless, next time I mentor I'll be trying to form a cohesive project
roadmap ahead of time.

## Find a suitable communication method

At Mozilla we use [IRC] a **lot**, but this is not a natural or perhaps
comfortable communication method for everyone. Perhaps because it's public, or
maybe because it can be intimidating to ask questions in a channel with dozens
of people in. Although both Justin and Ana did use IRC, I also made an effort to
find what other forms of communication they felt comfortable in. While Justin
and I used iMessage and Twitter DMs, Ana and I primarily used Hangouts and
WhatsApp.

## Check-in regularly

This is very much something I learned the hard way. While I set up a regular
video chat with Justin, I didn't do the same with Ana. With Justin in the
Mountain View office, it felt very natural and convenient for him to have a
scheduled chat, but Ana had a less robust internet connection, and the time
difference made it difficult to keep a schedule. This meant that, at times, a
few days passed where I wasn't sure if she was stuck, or simply working her way
through a task. Eventually I addressed this by just making sure that most days
I'd send Ana a short message on Hangouts to ask how she was doing. This let her
know that I was around if she had any problems she needed help with. I regret
not doing this from the start of the project.

## Keep notes

It really helps to keep good notes. This doesn't have to mean lots of notes
though! Keep a note of what is currently being worked on, who are the main
points of contact, what work is blocked and who can help to unblock it. Maintain
a list of what's next to work on so if something does come up and prevents
progress on one task, other tasks can be picked up. It's also important to
share these notes. For Justin we used [Etherpad], and as Ana's project was
entirely based in GitHub we mainly used the issue tracker. With hindsight, I
think a single place for Ana's project's notes would have been helpful too,
rather than spreading them across GitHub issues.

## Be open to new ideas and approaches

A personal challenge that I'm continuing to work on is to not have a fixed
solution in my head when I approach code reviews. I did have to stop myself from
falling into my default reactive tendency to be protective of my own code and
approaches to problem solving. By doing this, I open myself up to the creative
ways problems can be solved and learn about new techniques and libraries.

This also helps to avoid coming across overly critical in code reviews. That's
not to say you shouldn't have an idea how you might solve the problem -
understanding a potential solution will help to discuss the problem and to
give pointers if the participant gets stuck. If that does happen, although it is
tempting, try to avoid providing the full solution. Suggest they search for a
specific phrase, point them to a similar question online, or give them some
pseudo-code.

## Ask questions

Obvious questions would be checking in for status, but don't stop at that.
Perhaps ask them to explain why they took a particular approach, or ask what
else they tried. Ask if they're enjoying the work, and ask what you can do to
improve the experience. Don't just ask about the project though - I enjoyed
getting to know about Justin's entrepreneurial ventures, his favourite football
(soccer) team, and musical talents. I loved learning about Brazil from Ana, and
about her travels to Paris and Turin.

## Give feedback

This one is harder than it sounds: it's easy to say "you're doing great"
(especially if it's true), but it's not useful feedback. I need to continue
working on this one, but I feel that it's critically important to helping your
interns learn from their experience. Letting them know what they've done well,
how they've impressed you, and backing this up with specific examples has a lot
of value. The harder part is finding ways they can improve and bringing that
to them in a way they can absorb, and learn to grow from.

I really enjoyed being a mentor, and I am looking forward to having the
opportunity to do it again. If you're interested, you can watch both
[Justin's presentation] and [Ana's presentation] online.

Finally, I'd like to thank both Justin and Ana for being such awesome interns!

[Justin]: http://justinpotts.co/
[Ana]: http://anaplusplus.com/
[pytest-html]: https://github.com/pytest-dev/pytest-html
[CSP]: https://developer.mozilla.org/en-US/docs/Web/Security/CSP
[AMO]: https://addons.mozilla.org
[IRC]: https://wiki.mozilla.org/IRC
[Etherpad]: https://public.etherpad-mozilla.org/
[Justin's presentation]: https://air.mozilla.org/intern-presentations-2016-2/#@59m50s
[Ana's presentation]: https://air.mozilla.org/improving-pytest-html-my-outreachy-project/
