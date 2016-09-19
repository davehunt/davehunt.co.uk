---
layout: post
title: Drag and drop to object within a scrolling block
author: Dave Hunt
date: 2009-10-05 09:28:34 +0000
tags: selenium seleniumexamples.com
---
The `dragAndDropToObject` command works really well in Selenium, however it does
have some limitations. One such limitation I came across recently while writing
tests for an ExtJS web application: when your destination object is in a
scrolling box (and not in view) the command fails.<!--more-->

It appears that Selenium works out how far from the top of the screen the
destination object is, and attempts to drag the source object to this
destination even if it's way below the visible elements of the page. This
probably works fine for scrolling on a page wide scale, but for a block element
with `overflow:scroll` or `auto` it just doesn't work.

The following is a solution I put together in Java that works out all of the
dimensions, scrolls the parent block element so that the destination object is
in view, and then corrects the drag movement coordinates before issuing the
`dragAndDrop` command.

{% highlight java %}
//start coordinates
int startX = new Integer(selenium.getEval("this.getElementPositionLeft('id=sourceObject')"));
int startY = new Integer(selenium.getEval("this.getElementPositionTop('id=sourceObject')"));

//destination coordinates
int destinationX = new Integer(selenium.getEval("this.getElementPositionLeft('id=destinationObject')"));
int destinationY = new Integer(selenium.getEval("this.getElementPositionTop('id=destinationObject')"));

//destination dimensions
int destinationWidth = new Integer(selenium.getEval("this.getElementWidth('id=destinationObject')"));
int destinationHeight = new Integer(selenium.getEval("this.getElementHeight('id=destinationObject')"));

//scroll to destination
int destinationOffsetTop = new Integer(selenium.getEval("this.browserbot.findElement('id=destinationObjectContainer').offsetTop"));
selenium.getEval("this.browserbot.findElement('id=destinationObjectContainer').scrollTop = " + destinationOffsetTop);

//work out destination coordinates
destinationY = destinationY - destinationOffsetTop;
int endX = Math.round(destinationX + (destinationWidth / 2));
int endY = Math.round(destinationY + (destinationHeight / 2));
int deltaX = endX - startX;
int deltaY = endY - startY;
String movementsString = "" + deltaX + "," + deltaY;

selenium.dragAndDrop("id=sourceObject", movementsString);
{% endhighlight %}

There are likely to be limitations to this solution. A few I can think of:

 * What if the source object isn't in view due to a scrollbar on *it's* parent
   element?
 * What if the source or destination object has multiple ancestors with need to
   scroll them into view first?
 * What if the destination object's parent can't scroll to the top of the
   destination object - possible if it's the last child element?

Any suggestions of better ways to solve this problem?
