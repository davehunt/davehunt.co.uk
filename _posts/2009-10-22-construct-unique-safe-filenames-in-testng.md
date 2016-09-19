---
layout: post
title: Construct unique safe filenames in TestNG
author: Dave Hunt
date: 2009-10-22 11:07:23 +0000
tags: testng seleniumexamples.com
---
In order to save files for investigating failures from within TestNG it's
important to have a safe filename that is unique to the test - otherwise you may
overwrite important files. I have written the following simple method in Java
that is called from a listener with an `ITestResult` parameter to construct a
unique file name that should be safe on most file systems.<!--more-->

{% highlight java %}
private static String fileNameFrom(ITestResult result, String fileExtension) {
  List<String> parts = new ArrayList<String>();
  parts.add(result.getTestClass().getName());
  parts.add(result.getMethod().getMethodName());</p>

  //add parameters
  Object[] parameters = result.getParameters();
  for (Object parameter: parameters)
    parts.add(parameter != null ? parameter.toString() : NULL_PARAMETER_VALUE);</p>

  String fileName = StringUtils.join(parts.toArray(new String[parts.size()]), ".");
  fileName = fileName.replaceAll("[\\^\\\\.\\-:;#_]", "_");
  return fileName + "." + fileExtension;
}
{% endhighlight %}
