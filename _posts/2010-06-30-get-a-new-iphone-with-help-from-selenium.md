---
layout: post
title: Get a new iPhone with help from Selenium
author: Dave Hunt
date: 2010-06-30 12:59:08 +0000
tags: selenium iphone seleniumexamples.com
---
I've been looking forward to the iPhone 4 ever since the 3GS was released, as I
was in contract at the time and decided to wait another year for whatever Apple
decided to release next. Due to unexpected
([but very cool](http://www.meetup.com/seleniumsanfrancisco/calendar/13674328/))
circumstances, I was out of the country on the day Apple released the iPhone 4
and so was unable to queue up for one of the devices, and now due to the demand
it's very difficult to get hold of one. Since getting back to the UK I have been
visiting my two nearest O2 stores to check if they have had a delivery, and
today they pointed me towards their online stock checker...<!--more-->

So rather than check this manually, I decided to write a couple of small
Selenium 2 scripts to check the stock levels at my 'local' stores, and to find
out all stores that do have the devices. These examples use Java with TestNG
(for dataproviders), and Selenium 2.

# WebDriverTestBase.java

{% highlight java %}
package o2stock;

import java.util.concurrent.TimeUnit;

import org.openqa.selenium.WebDriver;
import org.openqa.selenium.firefox.FirefoxDriver;
import org.openqa.selenium.support.ui.Wait;
import org.openqa.selenium.support.ui.WebDriverWait;
import org.testng.annotations.AfterMethod;
import org.testng.annotations.BeforeMethod;

public class WebDriverTestBase {

    public static FirefoxDriver driver;
    public static Wait wait;

    @BeforeMethod(alwaysRun = true)
    protected void startWebDriver() {
    	driver = new FirefoxDriver();
    	wait = new WebDriverWait(driver, 60);
    	driver.manage().timeouts().implicitlyWait(5, TimeUnit.SECONDS);
    }

    @AfterMethod(alwaysRun = true)
    protected void closeSession() {
        driver.close();
    }

}
{% endhighlight %}

# O2Stock.java

{% highlight java %}
package o2stock;

import java.util.List;

import org.openqa.selenium.By;
import org.openqa.selenium.WebElement;
import org.openqa.selenium.support.ui.Select;
import org.testng.annotations.DataProvider;
import org.testng.annotations.Test;

public class O2Stock extends WebDriverTestBase {

    @SuppressWarnings("unused")
    @DataProvider
    private Object[][] stores() {
        return new Object[][] {
            { "London", "London" },
            { "South East", "Bluewater" },
            { "South East", "Dartford" },
        };
    }

    @Test(dataProvider = "stores")
    public void checkStock(String region, String city) {
        String modelTitle = "iPhone 4 32GB : Black";
        driver.get("http://stock.o2.co.uk/");
        driver.findElement(By.xpath("//input[following-sibling::label[@title='" + modelTitle + "']]")).click();
        driver.findElement(By.id("locRet")).click();
        new Select(driver.findElement(By.id("ukregions"))).selectByValue(region);
        new Select(driver.findElement(By.id("citySel"))).selectByValue(city);
        driver.findElement(By.xpath("//div[@class='sfSubmit']/input")).click();

        System.out.println(driver.findElement(By.className("timeStamp")).getText() + "\n");		

        int stores = driver.findElements(By.xpath("id('tblStyleRetail')/tbody/tr")).size();
        for (int i = 1; i <= stores; i++) {
            WebElement store = driver.findElement(By.xpath("id('tblStyleRetail')/tbody/tr[" + i + "]"));
            int withStock = store.findElements(By.xpath(".//img[@alt='In stock']")).size();
            System.out.println(withStock + " in stock at " + store.findElement(By.xpath("./td[1]")).getText());
        }
    }

    @SuppressWarnings("unused")
    @DataProvider
    private Object[][] models() {
        return new Object[][] {
            { "iPhone 4", "16GB", "Black" },
//          { "iPhone 4", "16GB", "White" },
            { "iPhone 4", "32GB", "Black" },
//          { "iPhone 4", "32GB", "White" },
            { "iPhone 3GS", "8GB", "Black" },
            { "iPhone 3GS", "16GB", "Black" },
            { "iPhone 3GS", "16GB", "White" },
            { "iPhone 3GS", "32GB", "Black" },
            { "iPhone 3GS", "32GB", "White" },
        };
    }

    @Test(dataProvider = "models")
    public void findStock(String model, String capacity, String colour) {
        String modelTitle = model + " " + capacity + " : " + colour;
        String modelName = model + " " + capacity + " (" + colour + ")";
        driver.get("http://stock.o2.co.uk/");
        driver.findElement(By.xpath("//input[following-sibling::label[@title='" + modelTitle + "']]")).click();
        driver.findElement(By.id("locRet")).click();
        new Select(driver.findElement(By.id("ukregions"))).selectByValue("All");
        new Select(driver.findElement(By.id("citySel"))).selectByValue("All");
        driver.findElement(By.xpath("//div[@class='sfSubmit']/input")).click();

        System.out.println(driver.findElement(By.className("timeStamp")).getText() + "\n");		

        List storesWithStock = driver.findElements(By.xpath("//img[@alt='In stock']/ancestor::tr"));
        if (storesWithStock.isEmpty()) {
            System.err.println(modelName + " is not in stock anywhere!\n");
            return;
        }

        System.out.println(modelName + " is in stock at the following " + storesWithStock.size() + " O2 stores:");
        for (WebElement store : storesWithStock) {
            System.out.println(store.findElement(By.xpath("./td[1]")).getText());
        }
        System.out.println();
    }

}
{% endhighlight %}

The current output of the `findStock` method for iPhone 4 is below:

<pre>Last update 01 Jul 2010 @ 13:00 - next update due around 17:00

iPhone 4 16GB (Black) is in stock at the following 5 O2 stores:
24 The Cascades, Cascade Centre, PORTSMOUTH, Hampshire, PO1 4RJ
98 Union Street, ABERDEEN, Scotland, AB11 6BD
Unit 3 Bon Accord Centre, George Street, ABERDEEN, Scotland, AB25 1HZ
219 Union Street, ABERDEEN, Scotland, AB11 6BA
26 South Walk, CWMBRAN, Wales, NP44 1PU</pre>

<pre>Last update 01 Jul 2010 @ 13:00 - next update due around 17:00

iPhone 4 32GB (Black) is in stock at the following 5 O2 stores:
24 The Cascades, Cascade Centre, PORTSMOUTH, Hampshire, PO1 4RJ
98 Union Street, ABERDEEN, Scotland, AB11 6BD
Unit 3 Bon Accord Centre, George Street, ABERDEEN, Scotland, AB25 1HZ
219 Union Street, ABERDEEN, Scotland, AB11 6BA
26 South Walk, CWMBRAN, Wales, NP44 1PU</pre>

Have fun!

**Update 1:** Some changes to the output, also including the datestamp of the
latest update. I have also made this available
[from my public svn repository](http://svn.blargon7.com/public/o2stock/).

**Update 2:** Also available on [GitHub](http://github.com/davehunt/o2stock).
