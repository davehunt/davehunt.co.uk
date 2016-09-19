---
layout: post
title: Cheesecake
author: Dave Hunt
date: 2010-11-07 19:08:55 +0000
tags: selenium cheesecake seleniumexamples.com
---
Here's another one of my odd Selenium examples! Last month I finally got to eat
at a 'Cheesecake Factory' restaurant, after first hearing about them a few years
back from an American friend. There's an overwhelming choice of 30+ cheesecakes
to choose from, and I'm really not good at making decisions when there's so much
choice. I decided to go with 'The Original', and that every subsequent visit to
the chain I would work my way down the menu, knowing I'd probably never get
through all of them in my lifetime!<!--more-->

So as I'm in California right now, I remembered my experience, and decided to
check out the full menu from the restaurant so that I could add it to Evernote
and keep track of my progress. Unfortunately I wasn't able to do a nice
copy+paste from the site, so - as usual - I wrote a Selenium script to list all
the cheesecakes. For a change this script uses the HtmlUnitDriver, however you
can easily swap in one of the other browser drivers.

{% highlight java %}
import org.openqa.selenium.By;
import org.openqa.selenium.WebElement;
import org.openqa.selenium.htmlunit.HtmlUnitDriver;
import org.testng.annotations.AfterTest;
import org.testng.annotations.BeforeTest;
import org.testng.annotations.Test;

import java.util.List;

public class CheesecakeFactory {

    HtmlUnitDriver driver;

    @BeforeTest
    public void startDriver() {
        driver = new HtmlUnitDriver();
    }

    @AfterTest
    public void stopDriver() {
        driver.close();
    }

    @Test
    public void listCheesecakes() {
        driver.get("http://www.thecheesecakefactory.com/");
        driver.findElement(By.linkText("Menu")).click();
        driver.findElement(By.linkText("Cheesecake")).click();
        List<WebElement> cheesecakes = driver.findElements(By.xpath("id('leftNav_levelTwo')//li"));

        System.out.println(cheesecakes.size() + " cheesecakes:");
        for (int i=0; i<cheesecakes.size(); i++) {
            System.out.println(i+1 + ". " + cheesecakes.get(i).getText());
        }
    }
}
{% endhighlight %}

<pre>34 cheesecakes:
1. The Original
2. Fresh Strawberry
3. Reeses Peanut Butter Chocolate Cake Cheesecake
4. 30th Anniversary Chocolate Cake Cheesecake
5. White Chocolate Raspberry Truffle®
6. Ultimate Red Velvet Cake Cheesecake™
7. Godiva® Chocolate Cheesecake
8. Fresh Banana Cream Cheesecake
9. Adam's Peanut Butter Cup Fudge Ripple
10. White Chocolate Caramel Macadamia Nut Cheesecake
11. Lemon Raspberry Cream Cheesecake
12. Dulce de Leche Caramel Cheesecake
13. Chocolate Coconut Cream Cheesecake
14. Tiramisu Cheesecake
15. Chocolate Mousse Cheesecake
16. Vanilla Bean Cheesecake
17. Chocolate Tuxedo Cream™ Cheesecake
18. Kahlua® Cocoa Coffee Cheesecake
19. Pineapple Upside-Down Cheesecake
20. Chocolate Raspberry Truffle®
21. Dutch Apple Caramel Struesel
22. Chocolate Chip Cookie - Dough Cheesecake
23. Wild Blueberry White Chocolate Cheesecake™
24. Low Carb Cheesecake
25. Low Carb Cheesecake with Strawberries
26. Key Lime Cheesecake
27. Caramel Pecan Turtle Cheesecake
28. Brownie Sundae Cheesecake
29. Snickers® Bar Chunks and Cheesecake
30. Craig's Crazy Carrot Cake Cheesecake
31. Oreo® Cheesecake
32. Cherry Cheesecake
33. Pumpkin Cheesecake
34. Pumpkin Pecan Cheesecake</pre>

Hmmm... Maybe I should get the Caltrain into San Francisco and make it 2 down,
32 to go!
