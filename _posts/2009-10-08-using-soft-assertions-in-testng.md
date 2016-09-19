---
layout: post
title: Using soft assertions in TestNG
author: Dave Hunt
date: 2009-10-08 12:39:43 +0000
tags: selenium testng seleniumexamples.com
---
One of the big differences between Selenium IDE and a Selenium RC solution is
the ability to perform 'soft' assertions. Selenium IDE users can append commands
with `verify` or `assert` to determine whether the test execution should stop
when a failure is observed. A popular use for this is to first assert that you
are on the correct page (`assertTitle`) and then verify elements on the page.
If you were only able to `assert` then your tests may fail early on, not
revealing further failures that may exist.<!--more-->

In Selenium RC you are limited by your test framework, and from my experience
using the Java client library there isn't a satisfactory equivalent to the soft
assertion feature of Selenium IDE. Some solutions propose that you catch the
assertions, log the occurrence of the failure, and check that there are no
failures at the end of the test. The problem here is remembering to check for
these verification failures.

Another solution suggests putting the check for verification failures in a
method that is run by the test framework after every test, however when these
fail (in TestNG) they are marked as configuration failures, and the default HTML
report can still report your test suites as passed.

Cédric Beust (creator of TestNG) [has discussed soft assertions on his blog](http://beust.com/weblog/archives/000514.html), but most of the proposed
solutions differ from the simple implementation that would encourage more
Selenium IDE users to adopt Selenium RC.

TestNG has support for custom listeners, which can run when tests
pass/fail/skip, as well as before and after invocation. By adding a custom
listener to check for verification failures after invocation, we can get the
details of all verification failures that have occurred, and report them at the
same time as we report our hard failure, or if there are no hard failures we can
change the result to a failure and report the verification failures.

This solution uses part of the TestNG soft failures patch by Dan Fabulich in
order to combine the stack traces of multiple failures.
[Details of the patch are available here](http://jira.opensymphony.com/browse/TESTNG-177).

I have created a simple Eclipse project that can be downloaded, extracted and
run. To keep it simple, this project does not use Selenium. You will need to add
these files into your own project.

# TestBase.java

This is the test base class, that imports TestNG and overrides the assert
methods so that test classes extending this class are decoupled from TestNG. I
have kept the methods in here to a minimum, in reality you will want to override
more of the assert methods and have more equivalent verify methods.

{% highlight java %}
package tests;

import java.util.ArrayList;
import java.util.HashMap;
import java.util.List;
import java.util.Map;

import org.testng.Assert;
import org.testng.ITestResult;
import org.testng.Reporter;

public class TestBase {

  private static Map> verificationFailuresMap = new HashMap>();

    public static void assertTrue(boolean condition) {
      Assert.assertTrue(condition);
    }

    public static void assertFalse(boolean condition) {
      Assert.assertFalse(condition);
    }

    public static void assertEquals(Object actual, Object expected) {
      Assert.assertEquals(actual, expected);
    }

    public static void verifyTrue(boolean condition) {
      try {
        assertTrue(condition);
      } catch(Throwable e) {
        addVerificationFailure(e);
      }
    }

    public static void verifyFalse(boolean condition) {
      try {
        assertFalse(condition);
      } catch(Throwable e) {
        addVerificationFailure(e);
      }
    }

    public static void verifyEquals(Object actual, Object expected) {
      try {
        assertEquals(actual, expected);
      } catch(Throwable e) {
        addVerificationFailure(e);
      }
    }

  public static List getVerificationFailures() {
    List verificationFailures = verificationFailuresMap.get(Reporter.getCurrentTestResult());
    return verificationFailures == null ? new ArrayList() : verificationFailures;
  }

  private static void addVerificationFailure(Throwable e) {
    List verificationFailures = getVerificationFailures();
    verificationFailuresMap.put(Reporter.getCurrentTestResult(), verificationFailures);
    verificationFailures.add(e);
  }

}
{% endhighlight %}

# TestListenerAdapter.java

This adapter implements TestNG's `IInvokedMethodListener` and overrides the two
methods `afterInvocation` and `beforeInvocation`. This means that when you
extend this class you only need to override the afterInvocation method.

{% highlight java %}
package tests;

import org.testng.IInvokedMethod;
import org.testng.IInvokedMethodListener;
import org.testng.ITestResult;

public class TestListenerAdapter implements IInvokedMethodListener {

	public void afterInvocation(IInvokedMethod arg0, ITestResult arg1) {}

	public void beforeInvocation(IInvokedMethod arg0, ITestResult arg1) {}

}
{% endhighlight %}

# CustomTestListener.java

This is where the verification failures are combined for the report, and
successful tests with verification failures are turned into failing tests.

{% highlight java %}
package tests;

import java.util.List;

import org.testng.IInvokedMethod;
import org.testng.ITestResult;
import org.testng.Reporter;
import org.testng.internal.Utils;

public class CustomTestListener extends TestListenerAdapter {

	@Override
	public void afterInvocation(IInvokedMethod method, ITestResult result) {

		Reporter.setCurrentTestResult(result);

		if (method.isTestMethod()) {

			List verificationFailures = TestBase.getVerificationFailures();

			//if there are verification failures...
			if (verificationFailures.size() > 0) {

				//set the test to failed
				result.setStatus(ITestResult.FAILURE);

				//if there is an assertion failure add it to verificationFailures
				if (result.getThrowable() != null) {
					verificationFailures.add(result.getThrowable());
				}

				int size = verificationFailures.size();
				//if there's only one failure just set that
        if (size == 1) {
          result.setThrowable(verificationFailures.get(0));
        } else {
          //create a failure message with all failures and stack traces (except last failure)
          StringBuffer failureMessage = new StringBuffer("Multiple failures (").append(size).append("):nn");
					for (int i = 0; i < size-1; i++) {
						failureMessage.append("Failure ").append(i+1).append(" of ").append(size).append(":n");
						Throwable t = verificationFailures.get(i);
						String fullStackTrace = Utils.stackTrace(t, false)[1];
						failureMessage.append(fullStackTrace).append("nn");
					}

					//final failure
					Throwable last = verificationFailures.get(size-1);
					failureMessage.append("Failure ").append(size).append(" of ").append(size).append(":n");
					failureMessage.append(last.toString());

					//set merged throwable
					Throwable merged = new Throwable(failureMessage.toString());
					merged.setStackTrace(last.getStackTrace());

					result.setThrowable(merged);
				}
			}
		}
	}

}
{% endhighlight %}

# VerifyTests.java

A few example tests that will all fail with verification or assertion failures.
Run these tests to see the differences in the HTML report.

{% highlight java %}
package tests;

import org.testng.annotations.Test;

public class VerifyTest extends TestBase {

	@Test
	public void test1() {
		verifyTrue(false);
		verifyEquals("pass", "fail");
		verifyFalse(true);
	}

	@Test
	public void test2() {
		verifyTrue(false);
		assertEquals("pass", "fail");
		verifyFalse(true);
	}

	@Test
	public void test3() {
		verifyTrue(true);
		verifyTrue(false);
		verifyTrue(true);
	}

	@Test
	public void test4() {
		assertTrue(true);
		assertTrue(false);
		assertTrue(true);
	}

}
{% endhighlight %}

[Click here to download]({{ site.github.url }}/assets/verify-demo.zip) the above
files, including dependencies, Eclipse project file, and Ant `build.xml` file.
