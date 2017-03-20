---
layout: post
title: Jenkins pipeline walkthrough
author: Dave Hunt
tags: [continuous integration, jenkins, pipelines]
comments: true
---
I've recently been migrating Mozilla's traditional Jenkins jobs for functional
UI testing of our web properties into [pipelines][]. The following describes
some of the common features of these pipelines. I've included my reasons for
these design choices, and highlighted limitations that I'm hoping will be
resolved as pipelines evolve. I've also written a post on
[my thoughts on Jenkins pipelines][thoughts on pipelines] and
[IRC notifications in Jenkins pipelines][irc notifications].

## Common practices

### Environment variables
We try to keep the required environment variables to a minimum, but the
following are used by all of our pipelines for functional UI testing of web
applications:

* **PYTEST_BASE_URL** - This is used by [pytest-base-url][] to identify the
  base URL of the application under test. It allows the target to be specified
  in the pipeline configuration in Jenkins, either using a parameter or
  hard-coded in the configuration.
* **SAUCELABS_USERNAME** - The username for our [Sauce Labs][] account, which
  is picked up by [pytest-selenium][]. We typically define this in the global
  environment variables for Jenkins, but it could be overridden for specific
  pipelines.
* **SAUCELABS_API_KEY** - The API key for our [Sauce Labs][] account. This is
  stored as text credential in Jenkins, and we use the
  [Credentials Binding plugin][] plugin to access it from our pipelines.

### Credentials
We use Jenkins credentials for securely storing text and file-based sensitive
data. These are then accessible to pipelines through the
[Credentials Binding plugin][].

### Deleting workspaces
In order to ensure we're starting from a known state, we wipe out the contents
of the workspace whenever we request a node using the `deleteDir()` step. This
prevents artefacts from previous builds being mistaken as results of the current
build. Our builds would most likely complete faster without this, however I
consider this additional time worthwhile to have confidence that you're working
against the correct workspace. Note that if you're using [docker][] or similar,
then this wouldn't be necessary.

### Timestamps
We're using the [Timestamper plugin][], which adds a timestamp next to each
entry in the console log. This can be useful when investigating build issues, or
identifying steps that are taking exceptionally long to execute. This plugin is
used by nesting your steps inside a `timestamps` step.

### ANSI colours
Some console output uses ANSI colours, such as the test outcome. By default,
Jenkins will not interpret these colours in the console log. We use the
[ANSIColor plugin][], and enable this in our pipelines by using the build
wrapper:

```groovy
node {
  wrap([$class: 'AnsiColorBuildWrapper']) {
    // other build steps go here
  }
}
```

This is an example of a plugin that doesn't have pipeline-specific syntax yet,
but it can still be used by knowing the Java class of the build wrapper. In
time, hopefully all plugins will have more memorable and readable pipeline
syntax.

### Timeouts
To prevent tests from stalling forever, we wrap the test command in a `timeout`
step, such as:

```groovy
timeout(time: 1, unit: 'HOURS') {
  // other build steps go here
}
```

## Checkout stage
The first stage is pretty self-explanatory. We clone the repository defined in
the pipeline job, and stash the workspace so that it can be un-stashed and used
by other steps in the pipeline without cloning multiple times.

```groovy
stage('Checkout') {
  node {
    timestamps {
      deleteDir()
      checkout scm
      stash 'workspace'
    }
  }
}
```

Our pipelines don't currently target specific nodes, so they could run on master
or any worker attached. I suspect this is something we're likely to change for
some stages, most likely using [docker][]. An example would be for running tests
against specific versions of Python.

## Linting stage
Before we run our tests, we check that the code meets our linting standards. For
most of our projects, this means running [flake8][] with any project-specific
overrides defined. As we're using [Tox][] to run this, the stage is pretty
simple. We just un-stash the workspace, and execute the appropriate Tox
environment:

```groovy
stage('Lint') {
  node {
    timestamps {
      deleteDir()
      unstash 'workspace'
      sh 'tox -e flake8'
    }
  }
}
```

## Tests stage
Some of our projects only execute tests against a single environment, however
it's more common to have multiple. For example, we may want to run the same
tests on multiple environments (operating system, browser, etc), or we may have
different suites for desktop and mobile. For this walkthrough I'll cover the
multiple-environment scenario.

### Desired capabilities
The functional UI tests for our Web projects use [Selenium](), and the way you
specify target environments and additional configuration is through desired
capabilities. Depending on your Selenium client and chosen framework, these
capabilities can be specified in different ways. We're using the official Python
client and [pytest-selenium][], which allows capabilities to be specified in a
file using a 'capabilities' key via [pytest-variables][]. In our pipelines we
have a `writeCapabilities` function, which accepts a map, and merges this with
some predefined defaults before creating a JSON file:

```groovy
import groovy.json.JsonOutput

def writeCapabilities(desiredCapabilities) {
    def defaultCapabilities = [
        build: env.BUILD_TAG,
        public: 'public restricted'
    ]
    def capabilities = defaultCapabilities.clone()
    capabilities.putAll(desiredCapabilities)
    def json = JsonOutput.toJson([capabilities: capabilities])
    writeFile file: 'capabilities.json', text: json
}
```

The `build` and `public` capabilities are specific to [Sauce Labs][], and we
provide suitable default values for these. The `env.BUILD_TAG` is a reference to
the `BUILD_TAG` environment variable provided by Jenkins, and allows a way for
us to identify specific builds in the Sauce Labs dashboard.

### Tox environments
We use a map to associate Tox environments with capabilities. The following
example demonstrates this for desktop and mobile test suites:

```groovy
def environments = [
  desktop: [
    browserName: 'Firefox',
    version: '47.0',
    platform: 'Windows 7'
  ],
  mobile: [
    browserName: 'Browser',
    platformName: 'Android',
    platformVersion: '5.0',
    deviceName: 'Android Emulator',
    appiumVersion: '1.5.3'
  ]
]
```

In our pipelines we pass the value for each environment to the
`writeCapabilities` function, which creates a `capabilities.json` file as
described above.

### Variables
To access application-specific variables such as credentials, we use the
`withCredentials` wrapper. These variables are stored as credentials files in
Jenkins, and need to be uploaded before your pipeline can use them:

```groovy
withCredentials([[
  $class: 'FileBinding',
  credentialsId: 'MY_VARIABLES',
  variable: 'VARIABLES']]) {
  withEnv(["PYTEST_ADDOPTS=--variables=${VARIABLES}"]) {
    runTox(environment)
  }
}
```

Here we also use the `withEnv` wrapper here to add the `--variables` command
line option to our pytest command.

### Additional pytest options
Options for [pytest][] are typically specified on the command line, however they
can also be defined via the `PYTEST_ADDOPTS` environment variable. In our
pipelines, we use `withEnv` to build the value for this variable. One example
command line option is the number of parallel processes to use with
[pytest-xdist][]. Here, we allow a `PYTEST_PROCESSES` variable to be used, but
default to 'auto'. I've slightly simplified the following example for
demonstration purposes:

```groovy
def processes = env.PYTEST_PROCESSES ?: 'auto'
withEnv(["PYTEST_ADDOPTS=${PYTEST_ADDOPTS} " +
  "-n=${processes} " +
  "--driver=SauceLabs " +
  "--variables=capabilities.json " +
  "--color=yes"]) {
  sh "tox -e ${environment}"
}
```

### Stashing results
Up until this stage, a failure would have caused the build to abort. In order to
collect our test results, we need to catch the exception thrown by a failure. We
then mark the build as failed, and stash the results for later use. In the
future it should be possible to avoid this, and instead use a post-build step
defined in the pipeline to gather and report the results. Once again, I've
simplified this following example for demonstration purposes:

```groovy
try {
  sh "tox -e ${environment}"
} catch(err) {
  currentBuild.result = 'FAILURE'
  throw err
} finally {
  dir('results') {
    stash environment
  }
}
```

Our tests are expected to write the results to a `results` subdirectory in the
workspace, which is defined in our `tox.ini`. We use the name of the Tox
environment to stash the results for later retrieval.

### Running environments in parallel
Pipelines allow steps to be run in parallel, which is perfect for our test
environments. In order to do this, we build a new map from our `environments`
map to pass to the `parallel` step. The following example has been simplified
for demonstration purposes:

```groovy
@NonCPS
def entrySet(aMap) {
  aMap.collect {
    k, v -> [key: k, value: v]
  }
}

def builders = [:]
for (entry in entrySet(environments)) {
  def environment = entry.key
  def capabilities = entry.value
  builders[(environment)] = {
    node {
      deleteDir()
      unstash 'workspace'
      writeCapabilities(capabilities)
      runTox(environment)
    }
  }
}

stage('Test') {
  parallel builders
}
```

The `entrySet` function is necessary for iterating over the `environments` map,
as this operation is not compatible with the [Groovy CPS][] that Jenkins
pipelines use. Fortunately, the workaround isn't too unsightly, but it is likely
to catch out a lot of people new to pipelines. More details of this issue can be
found [here][JENKINS-27421].

## Reporting
Currently, pipelines do not support post-build steps, so reporting of results
need to be in their own stage. This will be resolved by the
[Pipeline Model Definition plugin][], which is still in beta. As with
gathering the test results, it's necessary to catch an exception from the test
stage so that we always report the results:

```groovy
try {
  stage('Test') {
    parallel builders
  }
} catch(err) {
  currentBuild.result = 'FAILURE'
  ircNotification(currentBuild.result)
  mail(
    body: "${BUILD_URL}",
    from: "firefox-test-engineering@mozilla.com",
    replyTo: "firefox-test-engineering@mozilla.com",
    subject: "Build failed in Jenkins: ${JOB_NAME} #${BUILD_NUMBER}",
    to: "fte-ci@mozilla.com")
  throw err
} finally {
  stage('Results') {
    def keys = environments.keySet() as String[]
    def htmlFiles = []
    node {
      deleteDir()
      sh 'mkdir results'
      dir('results') {
        for (int i = 0; i < keys.size(); i++) {
          // Unstash results from each environment
          unstash keys[i]
          htmlFiles.add("${keys[i]}.html")
        }
      }
      publishHTML(target: [
        allowMissing: false,
        alwaysLinkToLastBuild: true,
        keepAll: true,
        reportDir: 'results',
        reportFiles: htmlFiles.join(','),
        reportName: 'HTML Report'])
      junit 'results/*.xml'
      archiveArtifacts 'results/*'
    }
  }
}
```

There's a lot going on here, but basically when we catch an exception from the
test stage we send an [IRC notification][irc notifications], and an email. Then,
regardless of whether there's been a failure, we create a `results` directory
and un-stash the results from each environment defined in our map. We're
expecting an HTML and XML report in each stash, and we pass details of these to
the [HTML Publisher plugin][] and `junit` steps. Finally, we archive the results
as artefacts.

This does mean that we only notify on test failures, so a failure in the
checkout or lint stage will not send any notifications. It would be possible to
address this by wrapping all of these stages in a try/catch, however this would
compromise the readability and maintainability of our pipelines. We're also not
currently sending notifications when a build is fixed. Again, this would be
possible by adding some logic around `currentBuild.result` but would make the
pipelines more complex. As these issues will be resolved by the
[Pipeline Model Definition plugin][], I'm inclined to accept these limitations
for now.

Here's a [full example][] of one of our Jenkins pipelines.

[pipelines]: https://jenkins.io/doc/book/pipeline/overview/
[thoughts on pipelines]: {{ site.github.url }}/continuous%20integration/2016/12/08/my-thoughts-on-jenkins-pipelines.html
[irc notifications]: {{ site.github.url }}/continuous%20integration/2016/11/25/irc-notifications-in-jenkins-pipelines.html
[pytest-base-url]: https://pypi.python.org/pypi/pytest-base-url/
[Sauce Labs]: http://saucelabs.com/
[pytest-selenium]: https://pypi.python.org/pypi/pytest-selenium/
[Credentials Binding plugin]: https://wiki.jenkins-ci.org/display/JENKINS/Credentials+Binding+Plugin
[docker]: https://www.docker.com/
[Timestamper plugin]: https://wiki.jenkins-ci.org/display/JENKINS/Timestamper
[ANSIColor plugin]: https://wiki.jenkins-ci.org/display/JENKINS/AnsiColor+Plugin
[flake8]: http://flake8.readthedocs.io/
[Tox]: http://tox.readthedocs.io/
[pytest-variables]: https://pypi.python.org/pypi/pytest-variables/
[pytest]: http://docs.pytest.org/
[pytest-xdist]: https://pypi.python.org/pypi/pytest-xdist/
[Groovy CPS]: https://github.com/cloudbees/groovy-cps
[JENKINS-27421]: https://issues.jenkins-ci.org/browse/JENKINS-27421
[Pipeline Model Definition plugin]: https://wiki.jenkins-ci.org/display/JENKINS/Pipeline+Model+Definition+Plugin
[HTML Publisher plugin]: https://wiki.jenkins-ci.org/display/JENKINS/HTML+Publisher+Plugin
[full example]: https://github.com/mozilla/Addon-Tests/blob/e683a4b96e646326956bb0e1611861e11fc02346/Jenkinsfile
