---
layout: post
title: Play Pacman with Selenium 2
author: Dave Hunt
date: 2010-05-25 17:09:19 +0000
tags: selenium pacman seleniumexamples.com
---
When I saw Google's recent
[interactive Pacman doodle](http://www.google.com/pacman/) to celebrate the
game's 30th anniversary, my first thought was 'Wouldn't it be cool to automate
playing Pacman using Selenium'. I know - I'm a geek!<!--more-->

![Google's Pacman Doodle]({{ site.github.url }}/assets/pacman.png)

Anyway, below is my quick proof of concept using Selenium 2 alpha 4  - the
latest version of Selenium 2 is available for download
[here](http://code.google.com/p/selenium/downloads/list).

{% highlight java %}
package tests;

import java.util.concurrent.TimeUnit;

import org.openqa.selenium.By;
import org.openqa.selenium.Keys;
import org.openqa.selenium.WebDriver;
import org.openqa.selenium.WebElement;
import org.openqa.selenium.firefox.FirefoxDriver;

public class WebDriverScratchPad {

	public static final long PAUSE = 2000;

	public static final Keys UP = Keys.ARROW_UP;
	public static final Keys DOWN = Keys.ARROW_DOWN;
	public static final Keys LEFT = Keys.ARROW_LEFT;
	public static final Keys RIGHT = Keys.ARROW_RIGHT;

	public static WebElement controls;

	public static void main(String [ ] args) throws InterruptedException {
		WebDriver driver = new FirefoxDriver();
		driver.manage().timeouts().implicitlyWait(20, TimeUnit.SECONDS);
		driver.get("http://www.google.com/pacman/");
		controls = driver.findElement(By.id("pcm-c"));

		move(LEFT, UP, LEFT, DOWN, LEFT, UP, LEFT, DOWN, RIGHT);
	}

	private static void move(Keys... directions) {
		for (Keys direction : directions) {
			controls.sendKeys(direction);
			try {
				Thread.sleep(PAUSE);
			} catch (InterruptedException e) {
				e.printStackTrace();
			}
		}
	}

}
{% endhighlight %}

Try it out for yourselves, and I extend these challenges to you:

 * Specify a distance for Pacman to travel rather than using an arbitrary pause
 * Detect if Pacman dies in his quest to eat all the pills
 * Work out the score at the end of the game

All of the above should be achievable... let me know how you get on. Also let me
know what high scores you achieve playing Pacman with Selenium 2!
