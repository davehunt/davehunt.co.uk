---
layout: post
title: Capturing screenshots from remote Selenium RC
author: Dave Hunt
date: 2009-10-16 12:11:31 +0000
tags: selenium seleniumexamples.com
---
Despite the name, the Selenium RC (*Remote* Control) server is often run on the
same machine as the testing framework, which makes saving screenshots to disk
quite easy. If however you are running Selenium RC on a separate machine, or are
using Selenium Grid it can become more difficult as the screenshots are also
saved on the remote machines.<!--more-->

To solve this you can use the `captureScreenshotToString` and
`captureEntirePageScreenshotToString` commands, which return a Base64 encoded
String of the screenshot, which you can then decode and save to disk on your
testrunner machine.

The following demonstrates the latter command (entire screenshot) in Java. I
have added this to a TestNG `afterInvocation` listener for failed tests so that
I have a screenshot of the page that resulted in the failure, which can be very
valuable for diagnosing issues.

{% highlight java %}
public void afterInvocation(IInvokedMethod method, ITestResult result) {
  if (!result.isSuccess()) {
    String imageName = "screenshot.png";
    String imagePath = outputDirectory + separator + imageName;
    try {
      String base64Screenshot = session().captureEntirePageScreenshotToString("");
      byte[] decodedScreenshot = Base64.decodeBase64(base64Screenshot.getBytes());
      FileOutputStream fos = new FileOutputStream(new File(imagePath));
      fos.write(decodedScreenshot);
      fos.close();
      Reporter.log("<a href=\"file:///" + imagePath + "\">Screenshot</a>");
    } catch (Exception e) {
      e.printStackTrace();
    }
  }
}
{% endhighlight %}

**Notes:**

 * This example uses `org.apache.commons.codec.binary.Base64`
 * In this example, `outputDirectory` is set in a `onStart` listener from the
   `ITestContext` parameter's `getOutputDirectory` method
 * In reality you'd want to construct `imageName` from the test method name so
   that it is unique for each test
 * The `captureEntireScreenshot*` commands have limited browser support.
   Currently I only capture these screenshots if the browser is Firefox
