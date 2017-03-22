---
layout: post
title: IRC notifications from Jenkins pipelines
author: Dave Hunt
date: 2016-11-25 11:50:26 +0000
tags: [continuous integration, jenkins, pipelines, irc]
comments: true
---
I've been migrating several of our Jenkins jobs to [pipelines][], and one of
the challenges was preserving our IRC notifications whenever a build result
changes. At this time, the [IRC plugin][] for Jenkins
[doesn't include support for pipelines][JENKINS-33922], however it is still
possible to trigger a notification using the `sh` step.<!--more--> The
following snippet connects to `irc.mozilla.org` and sends a build result notice
to `#fx-test-alerts`:

```groovy
def ircNotification(result) {
  def nick = "fxtest${BUILD_NUMBER}"
  def channel = '#fx-test-alerts'
  result = result.toUpperCase()
  def message = "Project ${JOB_NAME} build #${BUILD_NUMBER}: ${result}: ${BUILD_URL}"
  node {
    sh """
        (
        echo NICK ${nick}
        echo USER ${nick} 8 * : ${nick}
        sleep 5
        echo "JOIN ${channel}"
        echo "NOTICE ${channel} :${message}"
        echo QUIT
        ) | openssl s_client -connect irc.mozilla.org:6697
    """
  }
}
```

Due to the lack of pipeline support in the IRC plugin, it's not possible to
maintain a persistent connection to IRC for these notifications. This means
that if you have multiple builds finishing at the same time there may be
clashes in the registering of the bot's nick. This is why I've appended the
build number to the desired nick, which reduces this risk.

I've found it necessary to include a short sleep after sending the `NICK` and
`USER` commands, to avoid attempting to join the channel before the user is
registered. Ideally, we'd wait for the appropriate response for each of these
commands, but that's not really possible with this approach.

In order to trigger these notifications based on the result of the build, it is
currently necessary to use try/catch in your pipelines. This can add a lot of
extra and rather unpleasant lines to your script, as you may want a try/catch
around the entire build, specific stages, or even specific steps. I've decided
to make a few compromises in order to keep our pipelines easier to maintain, so
for now we'll only have IRC notifications if a specific stage fails.

```groovy
try {
  stage('Test') {
    runTests()
  }
} catch(err) {
  currentBuild.result = 'FAILURE'
  ircNotification(currentBuild.result)
  throw err
}
```

The [Pipeline Model Definition plugin][] provides a much improved way of
defining [notifications][], without the need for try/catch, and the ability to
trigger notification on various build results. I haven't experimented much with
this plugin as it's still in beta, but the syntax is encouraging. An IRC
notification on build result change should look something like:

```groovy
pipeline {
  agent label: 'master'
  stages {
    stage('Test') {
      runTests()
    }
  }
  post {
     change {
       ircNotification(currentBuild.result)
     }
  }
```

[pipelines]: https://jenkins.io/doc/book/pipeline/overview/
[IRC plugin]: https://wiki.jenkins-ci.org/display/JENKINS/IRC+Plugin
[JENKINS-33922]: https://issues.jenkins-ci.org/browse/JENKINS-33922
[Pipeline Model Definition plugin]: https://wiki.jenkins-ci.org/display/JENKINS/Pipeline+Model+Definition+Plugin
[notifications]: https://github.com/jenkinsci/pipeline-model-definition-plugin/wiki/Notifications
