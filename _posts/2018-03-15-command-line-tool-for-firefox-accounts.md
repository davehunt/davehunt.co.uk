---
layout: post
title: Command line tool for Firefox Accounts
slug: command-line-tool-for-firefox-accounts
date: '2018-03-15 10:27:39+00:00'
---
When testing services that depend on [Firefox Accounts][], it's useful to be able to create disposable test accounts. Fortunately we've had this ability from the very early days of the service, and our automated tests make heavy use of [PyFxA][] to create, verify, and ultimately destroying accounts. As useful as this is, it hasn't been particularly easy to create accounts for the purposes of manual testing. For the rare occasion that I've needed an account, I've either created them manually via main user interface with a disposable email account, or I've created a simple one-off script to create a batch of accounts. As I had this need again recently, I decided to write a simple command line tool for creating verified accounts and subsequently destroying them.<!--more-->

To use the tool, you'll need Python (despite a dependency not claiming to have support for Python 3, it's working fine for me using 3.6) and pip. It can then be installed by simply running `pip install fxacli`

Any account created by the tool will be stored in a JSON file named `.accounts` in the working directory. This allows for one or more accounts to be created, and then destroyed without the need to specify the target account.

To create an account:

```
$ fxacli create
Account created!
 - 🌐  https://api-accounts.stage.mozaws.net/v1
 - 📧  test-72a888a3f6@restmail.net
 - 🔑  IvOhSLzI
Account verified! 🎉
```

To destroy the most recently created account:

```
$ fxacli destroy
Account destroyed! 💥
 - 🌐  https://api-accounts.stage.mozaws.net/v1
 - 📧  test-72a888a3f6@restmail.net
 - 🔑  IvOhSLzI
```

To destroy all accounts created using the tool, pass the `--all` flag.

If you want to destroy an account not created by the tool, or if your `.accounts` file is for any reason unreadable, you can pass `--email` and `--password` options.

By default all accounts will be created/destroyed from the staging instance of Firefox Accounts. This is highly recommended, but if you know what you're doing, the target environment can be specified using the `--env` command line option.

Visit the [FxACLI package on PyPI][FxACLI] for links to the source code, issue tracker, and more.

[Firefox Accounts]: https://developer.mozilla.org/en-US/docs/Mozilla/Tech/Firefox_Accounts
[PyFxA]: https://github.com/mozilla/PyFxA
[FxACLI]: https://pypi.python.org/pypi/fxacli
