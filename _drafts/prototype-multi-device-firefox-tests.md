---
layout: post
title: Prototype multi-device Firefox tests
slug: prototype-multi-device-firefox-tests
---
With [Firefox Accounts][], you can access your tabs, history, and bookmarks from any device. You can even send tabs from one device to another, which is great when I find myself on a page that's not optimised for mobile, or if I get distracted at the weekend and find something I want to pick up when I get to work on Monday morning. While these features are awesome, I've had issues when the sync isn't triggered, or things don't go as expected. Some of these issues are known (and are being addressed), but currently it's too easy for regressions to be introduced.<!--more-->

Let's take the simple use case of saving a bookmark using Firefox on your phone, and later opening the bookmark on Firefox on desktop. In this scenario we have the mobile client, the [Firefox Accounts][] service, the [Sync][Firefox Sync] service, and the desktop client. We could be using [Firefox on Android][] or [iOS][Firefox on iOS] as our mobile client, and we could be using Firefox on macOS, Linux, or Windows. Other scenarios could involve multiple different mobile clients, such as syncing between a tablet and phone. There's a lot of configuration necessary, and many variations. Whilst each of the components have their own automated tests, there's currently no automation to take care of the basic end-to-end scenarios.

Part of the issue is that there are many individual components, and many ways they can be combined. Integration testing is currently carried out manually, which is time-consuming, and doesn't allow us to cover as many scenarios and device combinations as we'd like. Introducing automation to cover the basic scenarios will allow the testers more time to focus on exploration and more edge cases.

Over the last few weeks I've built a proof-of-concept test harness to automate end-to-end testing of multiple clients and server components. Initially I have focused on the previously mentioned scenario of syncing a bookmark from mobile to desktop, and limited to Firefox on iOS and macOS for now. Rather than create something entirely from scratch, I've brought together existing solutions for this initial prototype. This allowed me to pull something together relatively quickly, but does also bring some limitations and questions along.

### The Scenario

Let's remind ourselves of our initial scenario:

* Firefox on iOS:
  * Open a website
  * Save a bookmark
  * Sign into Firefox Accounts
  * Perform initial sync
* Firefox on macOS:
  * Sign into Firefox Accounts
  * Perform initial sync
  * Verify new bookmark exists

Finally, we'll need to present these results in a way the user can interpret, and can investigate in the event of a failure. For this we'll need a framework that can pull everything together, which is where I started.

### The Test Framework

My language of choice is Python, and my preferred test framework is [pytest][]. Being able to use something that I'm already familiar with for the framework allowed me to focus on the more challenging areas. By using pytest I'm also able to take advantage of several plugins I have built and maintain for other projects. Finally, if we decide in the future to land any part of this into mozilla-central, it shouldn't require too many changes as Python and pytest are already in use there.

### Generating Firefox Accounts

As a prerequisite, we need credentials for Firefox Accounts. Fortunately, we already have the [PyFxA][] package. This allowed me to create a pytest fixture to create accounts as needed and destroy them when they're done with. The following is a slightly simplified version of the fixture, which creates an account, verifies it, and ultimately destroys it once it's no longer needed:

```python
@pytest.fixture
def fx_account():
    account = TestEmailAccount()
    client = Client('https://api-accounts.stage.mozaws.net/v1')
    password = ''.join([random.choice(string.ascii_letters) for i in range(8)])
    session = client.create_account(account.email, password)
    account.fetch()
    message = account.wait_for_email(lambda m: 'x-verify-code' in m['headers'])
    session.verify_email_code(message['headers']['x-verify-code'])
    yield {'email': account.email, 'password': password}
    account.clear()
    client.destroy_account(account.email, password)
```

Whilst building out the prototype it was useful to have a Firefox Account that wasn't immediately destroyed, which is why I [built a command line tool for creating and destroying accounts][FxACLI].

### Automating Firefox on iOS

Fortunately there is already a suite of [automated UI tests for Firefox on iOS][], so I was able to build on the existing code. For our scenario I was able to get away with only making a few changes:

1. Created an `IntegrationTests.swift` file for the new script. Note that although this is technically a test itself, I'm referring to it as a script. This is because it only forms part of the overall integration test, and is essentially executed as a script. Of course, any failure encountered while running it will result in a test failure.
2. Added `LaunchArguments.StageServer` as a launch argument in  `BaseTestCase.swift` so the staging environment would be used for Firefox Accounts and Sync.
3. Switched from using `type` to `typeText` in `FxScreenGraph.swift` for the Firefox Accounts login screen. This allowed entry of characters not displayed on the initial keyboard layout. If we want to use the on screen keyboard, then we'll need to enhance the screen graph to support switching layouts. Alternatively, we could avoid using such characters.
4. Added the ability to set the timeout for `waitforExistence` as loading the Firefox Accounts login screen and performing the initial sync were occasionally taking longer than the default of 5 seconds.
5. Modified the existing `Fennec_Enterprise_XCUITests` scheme to include environment variables for the Firefox Account email and password so they can be used from the script.

With these changes, I was able to create the script to open a website and save it as a bookmark:

```swift
func testFxASyncBookmark () {
    // Go to a webpage, and add to bookmarks
    navigator.createNewTab()
    loadWebPage("www.example.com")
    navigator.nowAt(BrowserTab)
    bookmark()

    // Sign into Firefox Accounts
    navigator.goto(FxASigninScreen)
    waitforExistence(app.webViews.staticTexts["Sign in"], timeout: 10)
    userState.fxaUsername = ProcessInfo.processInfo.environment["FXA_EMAIL"]!
    userState.fxaPassword = ProcessInfo.processInfo.environment["FXA_PASSWORD"]!
    navigator.performAction(Action.FxATypeEmail)
    navigator.performAction(Action.FxATypePassword)
    navigator.performAction(Action.FxATapOnSignInButton)
    allowNotifications()

    // Wait for initial sync to complete
    waitforExistence(app.tables.staticTexts["Sync Now"], timeout: 10)
}
```

To run XCUITests from outside of Xcode, you need to use the `xcodebuild` command line tool. So, using FxACLI to create a test account, I can run my new script using the following commands:

```
$ fxacli create
Account created!
 - 🌐  https://api-accounts.stage.mozaws.net/v1
 - 📧  test-a478e06856@restmail.net
 - 🔑  CokFkuRA
Account verified! 🎉
$ export FXA_EMAIL=test-a478e06856@restmail.net FXA_PASSWORD=CokFkuRA
$ xcodebuild test -scheme Fennec_Enterprise_XCUITests -destination "platform=iOS Simulator,name=iPhone X" -only-testing:XCUITests/IntegrationTests/testFxASyncBookmark
--- snip ---
** TEST SUCCEEDED **
$ fxcli destroy
Account destroyed! 💥
```

In order to run the script from my pytest framework, I created a fixture named `xcodebuild`. This fixture patches the environment with the Firefox Account variables, and yields an `XCodeBuild` object:

```python
@pytest.fixture
def xcodebuild_log(pytestconfig, tmpdir):
    xcodebuild_log = str(tmpdir.join('xcodebuild.log'))
    pytestconfig._xcodebuild_log = xcodebuild_log
    yield xcodebuild_log

@pytest.fixture
def xcodebuild(fx_account, monkeypatch, xcodebuild_log):
    monkeypatch.setenv('FXA_EMAIL', fx_account.email)
    monkeypatch.setenv('FXA_PASSWORD', fx_account.password)
    yield XCodeBuild(xcodebuild_log)
```

The `XCodeBuild` object has a `test` method, which requires the identifier of the test to run. When the `test` method is called, the `xcodebuild` binary is started in a new process, and the output is redirected to a log file to later attach to the HTML report. The following test, although incomplete at this point, demonstrates running the XCUITest from pytest:

```python
def test_sync_bookmark_from_device(xcodebuild):
    xcodebuild.test('XCUITests/IntegrationTests/testFxASyncBookmark')
```

I noticed early on that once a test has signed into Firefox Accounts, the next test to run will remember the email address used and only ask for a password when attempting to sign in. There's likely a less expensive solution, but for now I've resolved this by running `xcrun simctl shutdown all` to shutdown all simulators, followed by `xcrun simctl erase all` to wipe them.

### Automating Firefox on Desktop

So far we've added a bookmark in Firefox on iOS and performed an initial sync. We now need to sign into Firefox Accounts on desktop Firefox, perform a sync, and verify the new bookmark is added. Like our UI tests for Firefox on iOS, we already have a solution for performing integration tests for Sync. It's called [TPS][], and I with the following tweaks I was able to get it to work for my prototype:

1. Added "mobile" bookmark folder to `bookmarks.jsm`, which is necessary to verify bookmarks in this location.
2. Removed attempt to load and validate the ping schema. The `_tryLoadPingSchema` method attempts to read a schema file from disk, which isn't present for my prototype so I've removed that, and another related code path.

The source code for TPS is stored in mozilla-central, and I don't want the entire repository to be a requirement for running my prototype. If we decide that TPS is the best approach for these tests, then we'd probably need to find a better way to share the code than my current approach of simply copying the extension source code and making local changes.

TPS works by launching Firefox with the extension installed and passing an argument to the test to run. This means it's necessary for us to write a TPS test in JavaScript for our scenario:

```javascript
EnableEngines(["bookmarks"]);

var phases = { "phase1": "profile1" };

// expected bookmark state
var bookmarksExpected = {
"mobile": [{
  uri: "http://www.example.com/",
  title: "Example Domain"}]
};

// sync and verify bookmarks
Phase("phase1", [
  [Sync],
  [Bookmarks.verify, bookmarksExpected],
]);
```

The phases allow Firefox to be restarted and for syncing to be performed across multiple profiles. Whilst this could be useful, I've currently enforced a single phase/profile for my prototype.

There's already a TPS test runner written in Python, so I was able to selectively pick what I needed for my prototype. I created several pytest fixtures that work together to package the TPS extension, configure a Firefox profile, and in a similar to our `xcodebuild` fixture, provide an interface for executing a test:

```python
@pytest.fixture(scope='session')
def tps_addon(tmpdir_factory):
    name = str(tmpdir_factory.mktemp('addon').join('tps'))
    shutil.make_archive(name, 'zip', os.path.join(here, 'tps'))
    os.rename('{}.zip'.format(name), '{}.xpi'.format(name))
    yield '{}.xpi'.format(name)

@pytest.fixture
def tps_config(fx_account):
    yield {'fx_account': {
        'username': fx_account.email,
        'password': fx_account.password}}

@pytest.fixture
def tps_log(pytestconfig, tmpdir):
    tps_log = str(tmpdir.join('tps.log'))
    pytestconfig._tps_log = tps_log
    yield tps_log

@pytest.fixture
def tps_profile(tps_addon, tps_config, tps_log):
    preferences = {
        'extensions.autoDisableScopes': 10,
        'extensions.legacy.enabled': True,
        'identity.fxaccounts.autoconfig.uri': urls['content'],
        'tps.config': json.dumps(tps_config),
        'tps.logfile': tps_log,
        'tps.seconds_since_epoch': int(time.time()),
        'xpinstall.signatures.required': False
    }
    yield Profile(addons=[tps_addon], preferences=preferences)

@pytest.fixture
def tps(pytestconfig, tps_log, tps_profile):
    yield TPS(pytestconfig.getoption('firefox'), tps_log, tps_profile)
```

I also added a required command line option for the path to Firefox. The TPS test runner actually downloads the latest Firefox nightly, which could be an option in the future.

The `TPS` object provided by the `tps` fixture provides a `run` method, which takes a test file as an argument. After adding this to our Python test we have something that looks like this:

```python
def test_sync_bookmark_from_device(tps, xcodebuild):
    xcodebuild.test('XCUITests/IntegrationTests/testFxASyncBookmark')
    tps.run('test_bookmark.js')
```

Now we have a working prototype that satisfies our scenario. Note that there's not much to see while the TPS test is running, however if you open the settings menu in Firefox you can watch the state transition from not signed in, to signed in, and the initial sync being performed. If you're really quick you can also see the mobile bookmark appearing in the menu.

### Running the Tests

To run the tests you will need to first follow the instructions for [building Firefox on iOS][]. You will also need to ensure you have [legacy Python][] (2.7) installed (there are dependencies that have not yet been updated to support modern Python). Finally, install [pipenv][], which will take care of the remaining Python dependencies.

You can then run the tests as follows, making sure you set the correct path to your Firefox binary:

```
$ cd python
$ pipenv install
$ pipenv run pytest --firefox=/path/to/Firefox.app/Contents/MacOS/firefox-bin
```

The tests will build and install the application to the simulator, which can cause a delay where there will be no feedback to the user. Also, note that each XCUITest that is executed will shutdown and **erase data from all available iOS simulators**. This assures that each execution starts from a known clean state.

### Reviewing Results

Hopefully you'll see something like the following in your console:

```
$ pipenv run pytest --firefox=/path/to/Firefox.app/Contents/MacOS/firefox-bin
============================= test session starts =============================
platform darwin -- Python 2.7.13, pytest-3.4.2, py-1.5.2, pluggy-0.6.0 -- /path/to/python2.7
cachedir: .pytest_cache
rootdir: /path/to/firefox-ios/python, inifile: pytest.ini
plugins: metadata-1.6.0, html-1.16.1, mozlog-3.7
collected 1 item

test_integration.py::test_sync_bookmark_from_device PASSED               [100%]

-------------- generated html file: /path/to/results/index.html ---------------
========================= 1 passed in 273.26 seconds ==========================
```

Even if the test fails, you should see the 'generated html file' somewhere in your console. Open this file in Firefox to review the results. If there was a failure, the report will include the details as shown in the console. You'll also find environment details including the version of desktop Firefox being used.

For each test there's a link to logs for TPS and xcodebuild. At the moment these are included regardless of the outcome, however they're mostly valuable for investigating failures. In the future we may decide to exclude them when tests pass.

### What's Next?

Well, the next thing is to gather feedback on this prototype. Does it make sense to use XCUITests, TPS, and pytest, or have I missed something that would improve the integration testing between multiple devices? If you've read this far, I suspect you have some opinions. Please get in touch by leaving a comment or finding me on IRC, Slack, Twitter, email, etc!

After I've given some time for feedback to reach me, we'll need to device how to rollout the prototype so that it can start to provide value. Initially we might start with a dedicated Mac Mini available via remote access to trigger the tests.

Then we'll need to review the test cases that we'd want to write. Ideally we'd keep the suite relatively small, as the tests will take some time to run. The idea is to cover the basic functionality where risk is high, and the more obscure scenarios will be covered by manual testing.

Other ideas for future plans include:

* Create a prototype for Firefox on Android. Perhaps we can reuse parts of this prototype, although we'd probably look into using some combination of UIAutomator, Espresso, and Appium for the Android automation parts.
* Experiment with Appium as an alternative to XCUITest. I went with XCUITest because we already had something in place, but perhaps there's some value in switching these tests to using Appium. It's at least worth investigating.
* Experiment with introducing WebDriver to the prototype. If we have a scenario that requires us to interact with either Firefox or web content (such as Firefox Accounts) then we may need to introduce WebDriver (Selenium).
* Look into setting up Continuous Integration for these tests. If the tests prove to be valuable and stable, then it would be ideal to run them whenever a new version of Firefox is avaiable. This could be a simple cron schedule, or something like a Jenkins agent running the tests when triggered.
* Create a prototype for Firefox on other devices. In the future we may have Firefox Accounts integration in Firefox for FireTV, so it would be great if we could include coverage here, too.

I've been pushing my code to a branch on my fork of Firefox on iOS. You can see a comparison between my branch and the upstream master [here][integration branch].

### Demo

If you're unable to run the tests locally, here's a recording of the test running along with reviewing the HTML report and logs:

{% youtube "https://www.youtube.com/watch?v=SBOLJQSOIjI" %}

[Firefox Accounts]: https://developer.mozilla.org/en-US/docs/Mozilla/Tech/Firefox_Accounts
[Firefox Sync]: https://wiki.mozilla.org/CloudServices/Sync
[Firefox on Android]: https://developer.mozilla.org/en-US/docs/Mozilla/Firefox_for_Android
[Firefox on iOS]: https://github.com/mozilla-mobile/firefox-ios
[pytest]: https://docs.pytest.org/
[PyFxA]: https://github.com/mozilla/PyFxA
[TPS]: https://developer.mozilla.org/en-US/docs/Mozilla/Projects/TPS_Tests
[building Firefox on iOS]: https://github.com/mozilla-mobile/firefox-ios#building-the-code
[legacy Python]: http://docs.python-guide.org/en/latest/starting/installation/#legacy-python-2-installation-guides
[pipenv]: https://docs.pipenv.org/
[integration branch]: https://github.com/mozilla-mobile/firefox-ios/compare/master...davehunt:integration
[FxACLI]: {{ site.baseurl }}{% post_url 2018-03-15-command-line-tool-for-firefox-accounts %}
