---
layout: post
title: Selenium 2 Examples
author: Dave Hunt
date: 2009-12-11 10:23:28 +0000
tags: selenium seleniumexamples.com
---
Yesterday [Selenium 2 (alpha 1) was released](http://groups.google.co.uk/group/selenium-developers/browse_thread/thread/ef3897dd08c1ab2e).
This is the first release since the Selenium and WebDriver projects started to
merge. The main difference is the inclusion of the WebDriver API into Selenium.
I've put together a small example below that uses the new API to log into two
web based e-mail clients and send an e-mail.<!--more-->

# WebDriverTestBase.java

{% highlight java %}
package tests;

import org.openqa.selenium.WebDriver;
import org.openqa.selenium.firefox.FirefoxDriver;
import org.openqa.selenium.support.ui.Wait;
import org.openqa.selenium.support.ui.WebDriverWait;
import org.testng.Assert;
import org.testng.annotations.AfterClass;
import org.testng.annotations.BeforeClass;

public class WebDriverTestBase {

	public static FirefoxDriver driver;
	public static Wait wait;

	@BeforeClass(alwaysRun = true)
    protected void startWebDriver() {
    	driver = new FirefoxDriver();
    	wait = new WebDriverWait(driver, 120);
    }

    @AfterClass(alwaysRun = true)
    protected void closeSession() {
	    driver.close();
    }

    public static void assertEquals(Object actual, Object expected) {
    	Assert.assertEquals(actual, expected);
    }

}
{% endhighlight %}

# VisibilityOfElementLocated.java

{% highlight java %}
package tests;

import org.openqa.selenium.By;
import org.openqa.selenium.WebDriver;
import org.openqa.selenium.support.ui.ExpectedCondition;

public class VisibilityOfElementLocated implements ExpectedCondition {

	By findCondition;

	VisibilityOfElementLocated(By by) {
		this.findCondition = by;
	}

	public Boolean apply(WebDriver driver) {
		driver.findElement(this.findCondition);
		return Boolean.valueOf(true);
	}

}
{% endhighlight %}

# WebmailTest.java

{% highlight java %}
package tests;

import org.openqa.selenium.By;
import org.testng.annotations.Test;

public class WebmailTest extends WebDriverTestBase {

	//variables
	public static final String YAHOO_EMAIL = "example@yahoo.co.uk";
	public static final String HOTMAIL_EMAIL = "example@hotmail.co.uk";

	@Test(description = "Sends an e-mail from Yahoo account")
	public void sendFromYahoo() {
		//new message variables
		String to = HOTMAIL_EMAIL;
		String subject = "Test Sending Email Message From Yahoo";
		String message = "This is a test e-mail from Yahoo";

		//login to yahoo
		driver.get("http://mail.yahoo.com/");
		driver.findElement(By.id("username")).sendKeys(YAHOO_EMAIL);
		driver.findElement(By.id("passwd")).sendKeys("mytestpw");
		driver.findElement(By.id(".save")).click();

		//create new message
		driver.findElement(By.id("compose_button_label")).click();
		wait.until(new VisibilityOfElementLocated(By.xpath("id('_testTo_label')/ancestor::tr[1]//textarea")));

		//send test message
		driver.findElement(By.xpath("id('_testTo_label')/ancestor::tr[1]//textarea")).sendKeys(to);
		driver.findElement(By.xpath("id('_testSubject_label')/ancestor::tr[1]//input")).sendKeys(subject);
		driver.switchTo().frame("compArea_test_");
		driver.findElement(By.xpath("//div")).sendKeys(message);
		driver.switchTo().defaultContent();
		driver.findElement(By.id("SendMessageButton_label")).click();
		//WARNING! sometimes a captcha is displayed here
		wait.until(new VisibilityOfElementLocated(By.xpath("//nobr[contains(text(), 'Message Sent')]")));
	}

	@Test(description = "Sends an e-mail from Hotmail account")
	public void sendFromHotmail() {
		//new message variables
		String to = YAHOO_EMAIL;
		String subject = "Test Sending Email Message From Hotmail";
		String message = "This is a test e-mail from Hotmail";

		//login to hotmail
		driver.get("http://mail.live.com/");
		driver.findElement(By.name("login")).sendKeys(HOTMAIL_EMAIL);
		driver.findElement(By.name("passwd")).sendKeys("mytestpw");
		if (driver.findElement(By.name("remMe")).isSelected()) {
			driver.findElement(By.name("remMe")).click();
		}
		driver.findElement(By.name("SI")).click();

		//create new message
		driver.switchTo().frame("UIFrame");
		driver.findElement(By.id("NewMessage")).click();

		//send test message
		driver.findElement(By.id("AutoCompleteTo$InputBox")).sendKeys(to);
		driver.findElement(By.id("fSubject")).sendKeys(subject);
		driver.switchTo().frame("UIFrame.1");
		driver.findElement(By.xpath("//body")).sendKeys(message);
		driver.switchTo().frame("UIFrame");
		driver.findElement(By.id("SendMessage")).click();
		assertEquals(driver.findElement(By.cssSelector("h1.SmcHeaderColor")).getText(), "Your message has been sent");
	}

}
{% endhighlight %}

Disclaimer: These tests are working at the time of this post but do require
active e-mail accounts. They are likely to fail when the web based e-mail
clients update their applications. Also, please don't misuse these examples -
their intention is to show how to use the WebDriver API and were inspired by a
test assignment for an interview process.
