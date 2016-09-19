---
layout: post
title: Checking Wedding Gifts with Selenium
author: Dave Hunt
date: 2010-08-23 10:28:33 +0000
tags: selenium seleniumexamples.com
---
Admittedly this one is unlikely to be useful to many, but it could still serve
as a useful example for Selenium 2. Also, I haven't posted an example since
[alpha 5](http://seleniumhq.wordpress.com/2010/07/14/selenium-2-0a5-released/)
was released.<!--more-->

I'm getting married in a few short weeks, and our gift list has now opened for
any guests that would like to buy us something for the occasion. We debated over
having a list in the first place, but in the end decided that it was probably
for the best. One of our hesitations was that we didn't want to know what people
had bought for us before the day, and having an online gift list makes it
possible to 'spoil' the surprise.

With the list only having been open a couple of days, the temptation was already
pretty difficult to resist and we decided that it was okay to see what has been
bought just so long as we don't find out who's bought what. Unfortunately this
isn't easy on the site as they're listed together, so I was sneakily able to
construct an XPath that extracted the necessary information. I've now combined
into a Selenium 2 example using Java, which can be found below.

{% highlight java %}
package giftlist;

import java.util.List;
import java.util.concurrent.TimeUnit;

import org.openqa.selenium.By;
import org.openqa.selenium.WebDriver;
import org.openqa.selenium.WebElement;
import org.openqa.selenium.firefox.FirefoxDriver;

public class GiftList {

	private static WebDriver driver;
	private static final String GIFT_LIST_NUMBER = "123546";
	private static final String GIFT_LIST_PASSWORD = "password";

	public static void main(String [ ] args) {
		//open browser
		driver = new FirefoxDriver();

		//set implicit wait
		driver.manage().timeouts().implicitlyWait(20, TimeUnit.SECONDS);

		//login
		driver.get("https://www.johnlewisgiftlist.com/");
		driver.findElement(By.linkText("Manage your list")).click();
		driver.findElement(By.name("giftListNumber")).sendKeys(GIFT_LIST_NUMBER);
		driver.findElement(By.name("listPassword")).sendKeys(GIFT_LIST_PASSWORD);
		driver.findElement(By.linkText("Display List")).click();

		//get purchased items
		driver.findElement(By.linkText("Gifts purchased")).click();
		driver.findElement(By.xpath("//h1[text()='Gifts purchased by guests']"));
		List<WebElement> purchasedItems = driver.findElements(By.xpath("//tr[td[2] and @class='item']"));

		//output purchased items
		for (WebElement item : purchasedItems) {
			System.out.println(item.findElement(By.xpath("/td[1]")).getText());
		}

		//log out
		driver.findElement(By.linkText("Log out")).click();

		//close browser
		driver.close();
	}
}
{% endhighlight %}
