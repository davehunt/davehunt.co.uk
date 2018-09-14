---
layout: post
title: Python unit tests now running with Python 3 at Mozilla
slug: python-unit-tests-now-running-with-python-3-at-mozilla
date: '2018-06-29 15:48:42+01:00'
---
I'm excited to announce that you can now run the Python unit tests for packages in the Firefox source code against Python 3! This will allow us to gradually build support for Python 3, whilst ensuring that we don't later regress. Any tests not currently passing in Python 3 are skipped with the condition `skip-if = python == 3` in the manifest files, so if you'd like to see how they fail (and maybe provide a patch to fix some!) then you will need to remove that condition locally. Once you've done this, use the `mach python-test` command with the new optional argument `--python`. This will accept a version number of Python or a path to the binary. You will need to make sure you have the appropriate version of Python installed.<!--more-->

Once you're ready to enable tests to run in [TaskCluster](https://docs.taskcluster.net/docs), you can simply update the `python-version` value in `taskcluster/ci/source-test/python.yml` to include the major version numbers of Python to execute the tests against. At the current time our build machines have Python 2.7 and Python 3.5 available.

To summarise:
1. Remove `skip-if = python == 3` from manifest files. These are typically named `manifest.ini` or `python.ini`, and are usually found in the `tests` directory for the package.
2. Run `mach python-test --python=3` with your target path or subsuite.
3. Fix the package(s) to support Python 3 and ensure the tests are passing
4. Add Python 3 to the `python-version` for the appropriate job in `taskcluster/ci/source-test/python.yml`.


At the time of writing, the [pythonclock.org](https://pythonclock.org/) tells me that we have just over 18 months before Python 2.7 will be retired. What this actually means is still somewhat unknown, but it would be a good idea to check if your code is compatible with Python 3, and if it's not, to do something about it. The Firefox build system at Mozilla uses Python, and it's still some way from supporting Python 3. We have a lot of code, it's going to be a long journey, and we could do with a bit of help!

Whilst we do plan to support Python 3 in the Firefox build system (see [bug 1388447](https://bugzilla.mozilla.org/show_bug.cgi?id=1388447)), my initial concern and focus has been the Python packages we distribute on the [Python Package Index (PyPI)](https://pypi.org/). These are available to use outside of Mozilla's build system, and therefore a lack of Python 3 support will prevent any users from adopting Python 3 in their projects. One such example is [Treeherder](https://github.com/mozilla/treeherder), which uses [mozlog](https://pypi.org/project/mozlog/) for parsing log files. Treeherder is a [django](https://www.djangoproject.com/) project, which recently dropped support for Python 2 (unless you're using their long term support release, which will support Python 2 until 2020).

Updating these packages to support Python 3 isn't necessary that hard to do, especially with tools such as [six](https://pythonhosted.org/six/), which provides utilities for handling the differences between Python 2 and Python 3. The problem has been that we had no way to run the tests against Python 3 in TaskCluster. This is no longer the case, and Python unit tests can now be run against Python 3!

So far I have enabled Python 3 jobs for our [mozbase](https://firefox-source-docs.mozilla.org/mozbase/index.html) unit tests (this includes the aforementioned mozlog), and our [mozterm](https://pypi.org/project/mozterm/) unit tests. There are still many tests in mozbase that are not passing in Python 3, so as mentioned above, these have been conditionally skipped in the manifest files. This will allow us to enable these tests as support is added, and this condition could even be used in the future if we have a package that doesn't have full compatibility with Python 2.

Now that running the tests against multiple versions of Python is relatively easy, it's a great time for me to encourage our community to help us with supporting Python 3. If you'd like to help, we have a [tracking bug](https://bugzilla.mozilla.org/show_bug.cgi?id=1093212) for all of our mozbase packages. Find a package you'd like to work on, read the comments to understand what you need and how to get set up, and let me know if you get stuck!
