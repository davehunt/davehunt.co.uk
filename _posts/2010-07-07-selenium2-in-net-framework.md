---
layout: post
title: Selenium 2 in .NET framework
author: Zac Campbell
date: 2010-07-07 17:05:02 +0000
tags: selenium .net seleniumexamples.com
---
I'm not Dave but you might have met me at one of the London Selenium meets. My
name is Zac and I have several years of automated testing experience with
Selenium RC in Java and C# and am now intending to be one of the early adopters
of Selenium 2 using the .NET version.

Selenium 2 and .NET are up to alpha4 already but thorough examples of code on
the internet are a bit thin on the ground so far compared to the Java
equivalent. In my time using Selenium RC and now Selenium 2 I have built up a
framework to support it.<!--more-->

I consider it to be an intermediate-level framework for using Selenium 2, NUnit
and the Page Object model together so some advanced users might find it a bit
basic. You may have even seen some of these concepts as snippets on testing
blogs around the place. But overall it offers a framework from which to develop
an easily maintainable suite of tests using the Page Object model (which is now
considered to be the best practice for Selenium 2). If you already have a suite
of Selenium tests in flat files that is spiralling in size and you are looking
to make it more durable and more efficient then this could suit you.

With this blog post I intend to step through and get you started with this
framework.

![The solution explorer for this example]({{ site.github.url }}/assets/solutionexplore.jpg)

# The Visual Studio solution

Exactly as we did with a Selenium RC project in Visual Studio, we need to create
a new 'Class Library' solution to work with. To get the same naming layout I
have, in the Name field type "Selenium2" and in the Solution Name field type
"Automated Test Suite".

The solution now needs references to both Selenium 2 and NUnit before we get
started so if you haven't already downloaded them then scurry off and do that
now.

Unzip the NUnit and Selenium 2 folders into your solution directory and add the
references to the files from inside VS so it looks like the screenshot. Even
though Selenium 2 is called Selenium 2, the reference files you need are
confusingly prefixed with WebDriver (WebDriver.common, etc)! But they definitely
are the ones you're after.

# SeleniumTestBase class

The class titled Selenium2TestBase.cs is used to setup and deliver the Selenium
2 driver to the test.

It's a good chance to set up your driver with any profile extensions or
configuration changes like bumping up the default timeout.

{% highlight csharp %}
using System.Configuration;
using OpenQA.Selenium;
using OpenQA.Selenium.Firefox;
using OpenQA.Selenium.IE;
using OpenQA.Selenium.Chrome;

namespace Selenium2
{
    public class Selenium2TestBase
    {
        private FirefoxProfile _ffp;
        private IWebDriver _driver;

        public IWebDriver StartBrowser()
        {
            Common.WebBrowser = ConfigurationManager.AppSettings["Selenium2Browser"];

            switch (Common.WebBrowser)
            {
                case "firefox":
                    _ffp = new FirefoxProfile();
                    _ffp.AcceptUntrustedCertificates = true;
                    _driver = new FirefoxDriver(_ffp);
                    break;
                case "iexplore":
                    _driver = new InternetExplorerDriver();
                    break;
                case "chrome":
                    _driver = new ChromeDriver();
                    break;
            }

            return _driver;
        }
    }
}
{% endhighlight %}

# Page Object GoogleHome.cs

The Page Object class contains all of the driver actions on that page as
methods. The aim is to keep each method concise and true to their purpose and
cut down on duplicated code. The Selenium2 driver is passed in from the test
when the class is instantiated. I have used static page objects that do not
return the driver. Alternatively you can use
[the technique covered here](http://code.google.com/p/selenium/wiki/PageObjects).
Also, [here's a Selenium RC example](http://www.theautomatedtester.co.uk/tutorials/selenium/page-object-pattern.htm).

To keep this example simple I have created only one Page Object but a typical
application will have several. I have also stored the Object's Xpath locators as
private readonly strings inside the Page Object in order to keep it simple and
concise but if you have a large amount of them to juggle you may want to use a
separate static class.

{% highlight csharp %}
using System;
using OpenQA.Selenium;

namespace Selenium2.Google
{
    class GoogleHome
    {
        private readonly string _inputFieldSearch = "//input[@name='q']";
        private readonly string _inputbuttonSearch = "//input[@name='btnG']";
        private readonly string _inputButtonLucky = "//input[@name='btnI']";

        private readonly IWebDriver googlehome;

        public GoogleHome(IWebDriver driver)
        {
            googlehome = driver;
            googlehome.Navigate().GoToUrl(Google_Common.GoogleTestSite);
        }

        public void TypeSearchString(string search){

            googlehome.FindElement(By.XPath(_inputFieldSearch)).SendKeys(search);
        }

        public void PressEnterOnSearchField()
        {
            googlehome.FindElement(By.XPath(_inputFieldSearch)).Submit();
         }
    }
}
{% endhighlight %}

# NUnit SetupFixture

This SetupFixture class (denoted by the NUnit tag [SetUpFixture]) runs before
the tests themselves. I mainly use this feature to configure Common variables
that are specific to this project. There is more information about how these are
managed further on in the post.

{% highlight csharp %}
using System;
using NUnit.Framework;

namespace Selenium2.Google
{
    [SetUpFixture]
    public class Google_Setup_Fixture
    {
        // Runs before all tests

        [SetUp]
        public void SetUp()
        {
            string environment = System.Configuration.ConfigurationManager.AppSettings["GoogleTestEnvironment"];
            Google_Common.GoogleTestSite = environment;

            // For example sometimes an environment is really slow
            // so we can raise the driver timeout for that env
            //if(environment == "dev")
            //    Common.DriverTimeout = new TimeSpan(0, 0, 90);
        }

        [TearDown]
        public void TearDown()
        {
            Google_Common.GoogleTestSite = null;
            Common.DriverTimeout = new TimeSpan(0, 0, 30);
            Common.WebBrowser = null;
        }
    }
}
{% endhighlight %}

# Finally, the test itself!

Now that we have the page actions (in Page Objects) and driver management
abstracted (split up) into their own classes we can actually write the test!

The TestFixture which contains the test keeps the familiar NUnit
TestFixtureSetUp and TestFixtureTearDown that you would have used in Selenium RC
to setup and teardown the browser, meaning that a new browser will be built for
each test. Note that my example inherits Selenium2TestBase, but you could easily
instantiate the class if you prefer.

The test itself uses the GoogleHome Page Object to do a basic search. The
details of the search are not important - I will presume that you know how to
write a test already.

But don't despair, we'll post more advanced uses of Selenium 2 in tests on the
SeleniumExamples blog as we use it more and more!

{% highlight csharp %}
using OpenQA.Selenium;
using NUnit.Framework;

namespace Selenium2.Google.Tests
{
    [TestFixture]
    public class Search_Tests : Selenium2TestBase
    {
        public IWebDriver driver;

        [TestFixtureSetUp]
        public void TestFixtureSetUp()
        {
            driver = StartBrowser();
        }

        [TestFixtureTearDown]
        public void TestFixtureTearDown()
        {
            driver.Quit();
        }

        [Test]
        public void SearchForSeleniumExamples()
        {
            string _expectedresult = "Selenium Examples";

            GoogleHome googlehome = new GoogleHome(driver);

            googlehome.TypeSearchString(_expectedresult);
            googlehome.PressEnterOnSearchField();

            //Assert that the title contains the search string
            Assert.That(driver.Title, Is.StringMatching(_expectedresult));

            // Check that we're still on the correct domain using the Google_Common class!
            Assert.That(driver.Url, Is.StringMatching(Google_Common.GoogleTestSite));
        }
    }
}
{% endhighlight %}

# Getting around an NUnit limitation

One thing that bothers me about NUnit is that I can't change the browser or test
environment from the NUnit GUI without having to rebuild the project. I guess
this is not a requirement for a unit test. Perhaps we're pushing the boundaries
using NUnit for cross browser functional testing.

However we can use a simple config file to call in variables, commonly the
browser, environment or whitelabel details of a site. In the XML config example
below I have key value pairs for both the browser and the base URL for the test
environment (ie dev, stage, live).

NUnit can be a bit sensitive to the config file's directory location. You can
see from the Solution Explorer screenshot that it is placed in the top folder
and named "Automated Test Suite.config". The settings in NUnit > Project >
Edit... must reflect the config file's location and name.

When you update the config file between tests you will need to reload the
project in the GUI (Ctrl-P) to use the config changes.

{% highlight xml %}
<?xml version="1.0" encoding="utf-8" ?>

<configuration>
  <configSections>
  </configSections>

  <appSettings>
    <!--<add key="Selenium2Browser" value="chrome"/>-->
    <add key="Selenium2Browser" value="iexplore"/>
    <!--<add key="Selenium2Browser" value="firefox"/>-->

    <add key="GoogleTestEnvironment" value="http://www.google.com/"/>
    <!--<add key="GoogleTestEnvironment" value="http://www.google.co.uk/"/>-->
  </appSettings>
</configuration>
{% endhighlight %}

# The Common classes
You may have noticed that the project references two 'Common' classes in
various places.

In the (static) Common class I store variables that are shared across the whole
Selenium2 namespace. For example here the TimeSpan value for timeouts is kept
and I can reference it from elsewhere. This way the driver and any custom
methods I have written will all use the same timeout value from the Common
class.

In other cases I might include the Common variables as part of the expected
criteria in assertions which cuts down on hard coded criteria.

{% highlight csharp %}
using System;

namespace Selenium2
{
    public static class Common
    {
        private static TimeSpan _defaultTimeSpan = new TimeSpan(0, 0, 30);

        public static string WebBrowser { get; set; }
        public static TimeSpan DriverTimeout
        {
            get { return _defaultTimeSpan; }
            set { _defaultTimeSpan = value; }
        }
    }
}
{% endhighlight %}

{% highlight csharp %}
namespace Selenium2.Google
{
    public static class Google_Common
    {
        public static string GoogleTestSite { get; set; }
    }
}
{% endhighlight %}

# Extending IWebDriver

Often you will find yourself writing tedious bits of code over and over again.
By adding an extension to IWebDriver you can call methods in the same manner,
keeping the code clean and readable.

{% highlight csharp %}
using System;
using NUnit.Framework;
using Selenium2;
using System.Threading;

namespace OpenQA.Selenium
{
    public static class IWebDriverExtensions
    {
        // This method finds a select element and then selects
        // the option element using the visible text
        public static void FindSelectElement(this IWebDriver driver, By bylocatorForSelectElement, String text)
        {
            IWebElement selectElement = driver.FindElement(bylocatorForSelectElement);
            selectElement.FindElement(By.XPath("//option[contains(text(), '" + text + "')]")).Select();
        }

        // This is a basic wait for element not present a'la Selenium RC
        // but sharing the same timeout value as the driver
        public static void WaitForElementNotPresent(this IWebDriver driver, By bylocator)
        {
            int timeoutinteger = Common.DriverTimeout.Seconds;

            for (int second = 0; ; second++)
            {
                Thread.Sleep(1000);

                if (second >= timeoutinteger) Assert.Fail("Timeout: Element still visible at: " + bylocator);
                try
                {
                    IWebElement element = driver.FindElement(bylocator);
                }
                catch (NoSuchElementException)
                {
                    break;
                }
            }
        }
    }
}
{% endhighlight %}

# Scaling Up!

With this solution it is quite easy to set up tests for numerous web apps within
the solution.

Add a new folder at the same level as the Google folder in my example and add
the Page Objects and tests inside it. Follow the same structure as the Google
example and both applications can share the Selenium2TestBase,
WebDriverExtensions, config and Common classes that you have already written.

Cutting down on duplicated code will save you tons of time in both writing and
maintaining your tests!
