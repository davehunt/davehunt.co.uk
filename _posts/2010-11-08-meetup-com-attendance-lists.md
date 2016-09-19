---
layout: post
title: Meetup.com Attendance Lists
author: Dave Hunt
date: 2010-11-08 11:53:55 +0000
tags: selenium meetup.com seleniumexamples.com
---
For the recent London Selenium Users Meetup Event I was asked if I could provide
the attendance list in a suitable format for creating labels for the guests when
they arrive. Given the short timeframe I did this simply by highlighting the
names on meetup.com, copying them, and pasting them into a simple text editor. I
then quickly cleaned this up before sending the list on. Within a short while
the list was out of date due to some members dropping out.<!--more-->

It did occur to me at the time that I could write a simple Selenium script, but
I didn't want to delay providing the list. Well I've now had some time to
revisit this, and hopefully for the next meetup event I'll be more prepared. The
following script should work on both upcoming and past events. It doesn't
support waiting lists, and may have problems if meetup.com truncates lists after
a certain number. I may revisit these items once I've scheduled the next event.

{% highlight java %}
import org.openqa.selenium.By;
import org.openqa.selenium.NoSuchElementException;
import org.openqa.selenium.WebElement;
import org.openqa.selenium.chrome.ChromeDriver;
import org.openqa.selenium.htmlunit.HtmlUnitDriver;
import org.testng.annotations.AfterTest;
import org.testng.annotations.BeforeTest;
import org.testng.annotations.Test;

import java.util.ArrayList;
import java.util.List;

public class Meetup {

    HtmlUnitDriver driver;

    String meetup = "seleniumlondon"; //meetup name from URL
    String event = "14712022"; //event ID from URL

    @BeforeTest
    public void startDriver() {
        driver = new HtmlUnitDriver();
    }

    @AfterTest
    public void stopDriver() {
        driver.close();
    }

    @Test
    public void listResponses() {

        //open the event page
        driver.get("http://www.meetup.com/" + meetup + "/calendar/" + event + "/");

        if (driver.findElement(By.xpath("id('C_document')/descendant::dl[@class='stats'][2]/dt")).getText().equals("Who's coming?")) {
            //this event has not yet occurred
            listMembers("id('C_document')//li[div[@class='D_attendeeHeader D_yes']]//li[starts-with(@id, 'member_')]", "Yes");
            listMembers("id('C_document')//li[div[@class='D_attendeeHeader D_maybe']]//li[starts-with(@id, 'member_')]", "Maybe");
            listMembers("id('C_document')//li[div[@class='D_attendeeHeader D_no']]//li[starts-with(@id, 'member_')]", "No");
        } else {
            //this event is in the past
            listMembers("id('C_document')//li[starts-with(@id, 'member_')]", "Attended");
        }
    }

    public void listMembers(String xpath, String label) {

        //get a list of all members that have responded or attended
        List<WebElement> memberElements = driver.findElements(By.xpath(xpath));

        int guestTotal = 0; //set guest total to zero
        List<Member> members = new ArrayList<Member>(); //create a list to store our members
        for (WebElement member : memberElements) {
            String name = member.findElement(By.className("D_name")).getText(); //member name

            int guests = 0; //set member's guests to zero

            //get the number of guests for this member
            try {
                //upcoming events use a different class for the guest count as past events
                String guestsTemp = member.findElement(By.className("guests")).getText();
                guests = new Integer(guestsTemp.substring(guestsTemp.indexOf("+")+1, guestsTemp.indexOf(" ")));
            } catch (NoSuchElementException e) { }

            try {
                String guestsTemp = member.findElement(By.className("D_guests")).getText();
                //past events use a different format for the guest count
                guests = new Integer(guestsTemp.substring(guestsTemp.indexOf("+")+1, guestsTemp.indexOf(")")));
            } catch (NoSuchElementException e) { }

            guestTotal = guestTotal + guests; //update the total number of guests for this event
            members.add(new Member(name, guests)); //add the current member to our list
        }

        //output a label for this list including total member and guest counts
        System.out.println("\n" + members.size() + " " + label + getGuestSuffix(guestTotal));

        for (int i=0; i<members.size(); i++) {
            //output details for each member
            System.out.println(i+1 + ". " + members.get(i).name + getGuestSuffix(members.get(i).guests));
        }
    }

    public String getGuestSuffix(int guests) {
        //return a suffix to indicate the number of guests
        if (guests > 0) {
            return " (+" + guests + " guest" + (guests > 1 ? "s" : "") + ")";
        } else {
            return "";
        }
    }

    //inner class for members
    public class Member {

        String name;
        int guests;

        public Member(String name, int guests) {
            this.name = name;
            this.guests = guests;
        }
    }
}
{% endhighlight %}
