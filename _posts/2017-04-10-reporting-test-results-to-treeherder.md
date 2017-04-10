---
layout: post
title: Reporting test results to Treeherder
author: Dave Hunt
tags: treeherder mozilla pulse jenkins
comments: true
---
Many of the web and services automated tests at Mozilla run in Jenkins, and until recently our instance was public. This meant it was easy for both paid and volunteer contributors to discover test failures, file issues, and provide fixes either for the tests or the projects they serve. Unfortunately, just like any software, Jenkins has had some security vulnerabilities. Last year, one of these prompted us to remove public access to our instance.<!--more-->

As Jenkins is much more than just a dashboard of results, a compromised instance could cause loss of data, or worse, remote execution of code. Since removing access to Jenkins, we've been looking into alternative ways to share our test results publicly.

Last month [I wrote about][activedata-post] how we're making the test results accessible via [ActiveData]. This allows anyone to query our results and perform data analysis and visualisation on them. Whilst this is going to prove to be a valuable tool, it doesn't provide a convenient dashboard for the results and is unlikely to attract much interest from the community.

The next step is submitting our results to [Treeherder] - a public dashboard for commits to Mozilla projects, which displays results of tasks such as builds, linting, and automated tests. Treeherder can be used to monitor the health of projects, and as a tool for investigating failures and raising defects, making it the perfect home for our test results.

The remainder of this post details the steps necessary to submit our test results to Treeherder. Note that whilst working on this I had local instances of most of the services and applications involved. This is a good practice as it reduces latency, and means you're not filling up an live instances with experimental test data.

## Creating a Pulse user
Treeherder ingests information about jobs from [Pulse] exchanges. Pulse is a RabbitMQ cluster, and if you're unfamiliar (as I was), I can highly recommend the [tutorials][rabbitmq-tutorials] to learn more. The first step was to sign into [PulseGuardian] and create a user for publishing our messages.

## Adding project repositories
To have our project repositories shown in Treeherder, it's necessary to [tell Treeherder about them][adding-github-repositories]. Then, to get result sets showing up you'll need to add [TaskCluster integration] for the repositories. For each revision, TaskCluster will send a message to Pulse, which Treeherder uses to build a collection of result sets for each repository. Note that you will need a GitHub organisation owner or repository administrator to enable TaskCluster integration. It's also worth noting that historic revisions will not be available in Treeherder, so only new commits will show up.

## Generating a message
Treeherder provides a [schema][treeherder-pulse-schema] that messages are expected to validate against. As we're using Jenkins, and have recently [migrated to declarative pipelines][declarative-pipelines-post] with a [shared library], I wrote a `submitToTreeherder` step to generate the payload using [json-schema-validator] to ensure we're conforming to the schema. Whilst developing locally I used a simple Python script based on the [RabbitMQ tutorials][rabbitmq-tutorials], as this made it much easier to iterate on the payload. Interestingly, this validation actually highlighted a couple of issues with the schema ([1352402][bug 1352402], [1352403][bug 1352403]). I also encountered issues with the Jenkins pipelines ([JENKINS-43195], [JENKINS-43246]).

## Submitting the message
Once the message has been generated and validated against the schema, we send it as our Pulse user to an exchange including the username with a routing key that contains the username and project. Our Pulse username is `fxtesteng`, so our exchange would be `exchange/fxtesteng/jobs`, and for the [FxAPOM] project our routing key would be `fxtesteng.fxapom`.

## Using Pulse Inspector
At this point Treeherder isn't aware of the new exchange, but you can use [Pulse Inspector] to listen on a specified exchange and routing key pattern to see the messages as they're received by Pulse. If you want to listen to all messages published to the Firefox Test Engineering exchange for Treeherder jobs, enter the exchange `exchange/fxtesteng/jobs` and click 'Start Listening'. Note that traffic on this exchange is currently very low, but it's useful if you've triggered a job and want to see what's being sent.

## Registering with Treeherder
Now that we're sending messages for each job, all that's left is to [tell Treeherder about your exchange][register-with-treeherder]. Once that's done, the next job published to Pulse should be picked up by Treeherder and displayed under the appropriate repository. The following screenshot shows FxAPOM results in Treeherder.

![FxAPOM results in Treeherder](/assets/treeherder-fxapom.png)

You can [see these for yourself][fxapom-treeherder], and take a look at the log files, HTML reports and other details for the results.

## What's next?
At the time of writing, only FxAPOM is configured to submit results to Treeherder. This repository has a low volume of commits, and the tests are run once a day. Next, we'd like to submit results for more of our repositories. As we're using our shared library, this is a relatively small change for most of our projects. There are also [a number of enhancements][fxtest-treeherder-issues] filed for the shared library, which will improve the display of the results in Treeherder. If you're interested in working on any of these, please add a comment and I'd be happy to mentor you.

I'd like to think that this model could be repeated for other continuous integration services. It's already provided by TaskCluster, but there's certainly some value in submitting results from [Travis CI] or [CircleCI]. That said, there are no current plans to implement this (that I'm aware of). If this is something you'd like to work on, let me know!

## Acknowledgements
I would like to say thanks to the Treeherder team for their assistance and considerable patience whilst I worked on this.
I'd also like to thank [Andrew Bayer] from the Jenkins team for his encouragement and assistance while I battled through a few Groovy and Jenkins pipeline issues. Whenever we're in the same city, I definitely owe that guy a drink!

[ActiveData]: https://wiki.mozilla.org/EngineeringProductivity/Projects/ActiveData
[Treeherder]: https://wiki.mozilla.org/EngineeringProductivity/Projects/Treeherder
[Pulse]: https://wiki.mozilla.org/Auto-tools/Projects/Pulse
[rabbitmq-tutorials]: http://www.rabbitmq.com/getstarted.html
[PulseGuardian]: https://pulseguardian.mozilla.org/
[add-repos-to-treeherder]: http://treeherder.readthedocs.io/submitting_data.html#adding-a-github-repository
[treeherder-pulse-schema]: https://github.com/mozilla/treeherder/blob/master/schemas/pulse-job.yml
[adding-github-repositories]: http://treeherder.readthedocs.io/submitting_data.html#adding-a-github-repository
[TaskCluster integration]: https://github.com/integration/taskcluster
[json-schema-validator]: https://github.com/daveclayton/json-schema-validator
[shared library]: https://github.com/mozilla/fxtest-jenkins-pipeline
[bug 1352402]: https://bugzilla.mozilla.org/show_bug.cgi?id=1352402
[bug 1352403]:https://bugzilla.mozilla.org/show_bug.cgi?id=1352403
[JENKINS-43195]: https://issues.jenkins-ci.org/browse/JENKINS-43195
[JENKINS-43246]: https://issues.jenkins-ci.org/browse/JENKINS-43246
[FxAPOM]: https://github.com/mozilla/fxapom
[Pulse Inspector]: https://tools.taskcluster.net/pulse-inspector/
[register-with-treeherder]: file:///Users/dhunt/workspace/treeherder/docs/_build/html/submitting_data.html#register-with-treeherder
[fxapom-treeherder]: https://treeherder.allizom.org/#/jobs?repo=fxapom&revision=cd87f7f8bb06ff035ac716081e01a6f55046911d
[fxtest-treeherder-issues]: https://github.com/mozilla/fxtest-jenkins-pipeline/labels/treeherder
[Travis CI]: https://travis-ci.org/
[CircleCI]: https://circleci.com/
[Andrew Bayer]: https://github.com/abayer
[activedata-post]: {% post_url 2017-03-21-analysing-pytest-results-using-activedata %}
[declarative-pipelines-post]: {% post_url 2017-03-23-migrating-to-declarative-jenkins-pipelines %}
