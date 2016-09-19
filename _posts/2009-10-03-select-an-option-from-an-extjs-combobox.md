---
layout: post
title: Select an option from an ExtJS ComboBox
author: Dave Hunt
date: 2009-10-03 13:55:03 +0000
tags: selenium extjs seleniumexamples.com
---
Automating an ExtJS web application can be difficult due to the dynamic nature
of the page. For example, the majority of unique ID attributes in the HTML will
be different between builds, which causes problems locating elements reliably.
Another issue is selected items from a ComboBox, which is not a normal HTML
`<select>` element, but an `<input>` that is populated from data in a completely
separate section of the DOM.<!--more-->

Below is a solution for locating the ComboBox elements and the Selenium commands
needed to select values in them. This example is taken from code written for the
Selenium RC Java client library.

{% highlight java %}
//click the down arrow image on the right of the ComboBox and assumes that there is a label before the component
selenium.click("//label[text()='My Combo List']/following-sibling::div/descendant::img[contains(@class, 'x-form-arrow-trigger')]");

//wait for a drop down list of options to be visible
selenium.waitForCondition("var value = selenium.isElementPresent('//div[contains(@class, 'x-combo-list') and contains(@style, 'visibility: visible')]'); value == true", "60000");

//click the required drop down item based on the text of the target item
selenium.click("//div[contains(@class, 'x-combo-list')]/descendant::div[contains(@class, 'x-combo-list-item')][text()='My Value']");

//wait for the drop down list of options to be no longer visible
selenium.waitForCondition("var value = selenium.isElementPresent('//div[contains(@class, 'x-combo-list') and contains(@style, 'visibility: visible')]'); value == false", "60000");
{% endhighlight %}
