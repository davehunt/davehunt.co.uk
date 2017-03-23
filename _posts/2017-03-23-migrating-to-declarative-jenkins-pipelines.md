---
layout: post
title: Migrating to declarative Jenkins pipelines
author: Dave Hunt
tags: [continuous integration, jenkins, pipelines, declarative]
comments: true
---
Last year I shared [my thoughts on Jenkins pipelines][thoughts on pipelines]
and provided a [walkthrough][jenkins pipeline walkthrough] of how we're using
pipelines at Mozilla. Since then, the [Pipeline Model Definition plugin] has
came out of beta, and we've been migrating our pipelines to the new declarative
syntax with a [shared library][shared libraries].<!--more-->

## Shared libraries
As soon as you're implementing similar steps in multiple Jenkins pipelines, it
makes sense to consider writing a shared library. Our [fxtest-jenkins-pipeline]
library allows us to centrally maintain custom steps such as IRC notifications,
or creating variables files with desired capabilities for Selenium, rather
than implementing these in every pipeline.

## Pipeline options
In our original pipelines it was necessary to wrap steps in order to configure
timeouts, ANSI colours, and timestamps. With declarative this is made so much
better by allowing [pipeline-specific options][pipeline options] to be
configured. The following snippet demonstrates enabling all three of these:

```groovy
options {
  ansiColor('xterm')
  timestamps()
  timeout(time: 1, unit: 'HOURS')
}
```

Everything defined in one place, and with less nesting/indentation vastly
improves the readability and maintainability of the pipelines.

## Environment variables
Another huge improvement is the handling of
[environment variables][pipeline environment] variables and accessing
credentials. Previously it was necessary to wrap steps in `withEnv` for
environment variables, and `withCredentials` for credentials. This is what we
previously would have needed:

```groovy
withCredentials([[
  $class: 'StringBinding',
  credentialsId: 'SAUCELABS_API_KEY',
  variable: 'SAUCELABS_API_KEY']]) {
  withEnv(["PYTEST_ADDOPTS=" +
    "-n=${processes} " +
    "--driver=SauceLabs " +
    "--variables=capabilities.json " +
    "--color=yes"]) {
      // ...
}
```

With declarative, this can be replaced with the following:

```groovy
environment {
  PYTEST_ADDOPTS =
    "-n=10 " +
    "--tb=short " +
    "--color=yes " +
    "--driver=SauceLabs " +
    "--variables=capabilities.json"
  SAUCELABS_API_KEY = credentials('SAUCELABS_API_KEY')
}
```

Note that there's a [regression][JENKINS-42857] in version 1.1.1 of the plugin
that prevents strings from being split over multiple lines, but this is already
fixed and will be included in the next release.

## Post build steps
Possibly the most valuable addition to declarative piplines are the post build
steps. It's now possible to define steps to execute after each stage or
pipeline depending on the current status of the build. Previously we were using
try/catch/finally for our tests step ensure we reported our results, and a
broader try/catch to send failure notifications.

We now have a `post` section immediately after our test stage that publishes
artifacts, similar to the following snippet:

```groovy
stage('Test') {
  steps {
    // ...
  }
  post {
    always {
      archiveArtifacts 'results/*'
      junit 'results/*.xml'
    }
  }
}
```

We also have a `post` section for the entire pipeline for notifications. This
means that we send notifications when any stage fails, which is another
improvement over our previous pipelines.

## Conclusion
In my opinion declarative pipelines are a huge improvement over the original
scripted pipelines. The new syntax is succinct and easier to read and maintain.
As a result of migrating to declarative and our shared library, one of our
pipelines had a 60% reduction in the lines of code but with more functionality!

Here's a [full example] of one of our declarative Jenkins pipelines.

[thoughts on pipelines]: {% post_url 2016-12-08-my-thoughts-on-jenkins-pipelines %}
[jenkins pipeline walkthrough]: {% post_url 2016-12-19-jenkins-pipeline-walkthrough %}
[Pipeline Model Definition plugin]: https://wiki.jenkins-ci.org/display/JENKINS/Pipeline+Model+Definition+Plugin
[shared libraries]: https://jenkins.io/doc/book/pipeline/shared-libraries/
[fxtest-jenkins-pipeline]: https://github.com/mozilla/fxtest-jenkins-pipeline
[pipeline options]: https://jenkins.io/doc/book/pipeline/syntax/#options
[pipeline environment]: https://jenkins.io/doc/book/pipeline/syntax/#environment
[full example]: https://github.com/mozilla/fxapom/blob/0f3b3cdb161940614ef50f2203a4633df1464c74/Jenkinsfile
[JENKINS-42857]: https://issues.jenkins-ci.org/browse/JENKINS-42857
