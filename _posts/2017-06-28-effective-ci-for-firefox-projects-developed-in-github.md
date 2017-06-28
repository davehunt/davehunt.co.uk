---
layout: post
title: Effective CI for Firefox projects developed in GitHub
author: Dave Hunt
tags: [firefox, continuous integration, github]
comments: true
---
Whilst the [canonical repository](https://hg.mozilla.org/) for the Firefox source code uses [Mercurial](https://www.mercurial-scm.org/), it's becoming increasingly popular for Firefox projects to use [GitHub](https://github.com/) for development. When it's time to ship, many of these projects will land their code inside the canonical repository for inclusion in the upcoming Firefox release. There are a few challenges that come with this approach.<!--more-->

Primarily, we won't know when a change in the project introduces a regression in Firefox until we attempt to land the project alongside the Firefox source code. Similarly, we won't know when a change to Firefox might cause a regression in the project.

This quarter I spent some time discussing these issues, learning how different teams are addressing the problems, and helping to define a more generic approach for the future.

## Current solutions
Here's a summary of some the solutions I discovered teams are using:

* [ActivityStream](https://github.com/mozilla/activity-stream) are adding their project to a local copy of the Firefox source code and pushing to an alternate branch. This then runs all the tests and reports results to [Treeherder](https://treeherder.mozilla.org/#/jobs?repo=pine). This relies on access to the alternate branch, and is currently a manual process.
* [Normandy](https://github.com/mozilla/normandy) download an archive from [gecko-dev](https://github.com/mozilla/gecko-dev) (a read-only Git mirror of the Firefox repository), combine this with the project repository, build Firefox, and run project specific tests. Every two weeks or so, they bundle the changes up and push to the Firefox repository.
* [debugger.html](https://github.com/devtools-html/debugger.html) use a Docker image with the Firefox source code, and run project specific tests in [Circle CI](https://circleci.com/).

## Proposed solution
Whilst I wasn't able to pull together a prototype this quarter, I did draft a proposal based on my investigations:

* Each project would create an entry script `mcmerge.sh`, which would be responsible for integrating the project source code with the Firefox source code. In some instances this might simply copy files, but it could also apply patches as needed.
* Whenever a pull request is opened against the GitHub project, or a change is pushed:
   * Check if the author of the pull/push is authorised to trigger continuous integration. This might be that they're a repository collaborator, or have authority to push to the main Firefox repository. If they are not authorised, we stop here.
 * Prepare a new patch using the [vcssync](https://bugzilla.mozilla.org/show_bug.cgi?id=1357597) tool currently in development, which will take the entire GitHub project repository at the current revision and apply it in the patch to a vacant subdirectory in the Firefox repository.
 * Push the prepared patch to our [Try Server](https://wiki.mozilla.org/Try), which will trigger tests according to the commit message. This syntax should be configurable in the project repository, but would likely default to all platforms/tests.
 * If triggered by a pull request, publish a comment indicating that the tests have been triggered and linking to the results in Treeherder.
 * During the regular Firefox build, we'd scan for these projects and execute any `mcmerge.sh` scripts we encountered. This would integrate the project with the Firefox source code.
 * Build Firefox and execute the tests as usual.
 * Reviewers of the pull request would be expected to check the results of the Try push as part of their review.

I also identified a few possible directions this may evolve in the future:

* We could use the same mechanism to automatically land changes that are pushed to the GitHub repository into the Firefox repository. This would mean that the Firefox repository would effectively hold a read-only mirror of the GitHub project. This would remove the need for manually pushing the project to the Firefox repository, and would allow us to determine when changes to the Firefox source code causes regressions in the GitHub projects. This may not be desirable for all projects, and would have implications for the sheriffs, who may need to back out changes that cause bustage.
* Rather than perform full builds of Firefox, we may be able to optimise these in a similar fashion to the artefact builds. This isn’t something we can do now, but it is a desirable build optimisation for other reasons.
* Due to the intermittent nature of our tests, it’s not currently possible to give a pass/fail indication in the GitHub pull request. If the scope of the tests is narrow enough and they’re stable, it's possible we could enhance this to poll for the result and indicate the outcome in the pull.

## Next steps
As mentioned, I was unfortunately not able to bring together a prototype based on my proposal. I discovered early on that I wasn't the only one looking into this, and I even feel that the other work in this area is already in better hands. The work in [bug 1364561](https://bugzilla.mozilla.org/show_bug.cgi?id=1364561) and [bug 1357597](https://bugzilla.mozilla.org/show_bug.cgi?id=1357597) are particularly relevant to this project. It's possible that I'll return to this project in the future, but for now I'll be following the progress contributing to the discussions without actively working on the implementation.
