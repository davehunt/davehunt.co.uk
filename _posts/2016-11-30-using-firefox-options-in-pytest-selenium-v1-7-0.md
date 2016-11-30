---
layout: post
title: Using Firefox options in pytest-selenium v1.7.0
author: Dave Hunt
categories:
- selenium
tags:
- selenium
- pytest
- firefox
comments: true
---
I'm please to announce that pytest-selenium v1.7.0 has been released, and
there's a change to the way Firefox options are handled. Previously, the profile
and binary path for Firefox were passed as keyword arguments when instantiating
the Firefox driver. Now, this is handled by an `Options` object.

This just means that the creation of the driver object when using Firefox has
gone from look something like this:

```python
driver = Firefox(
    capabilities=capabilities,
    executable_path=driver_path,
    firefox_binary=FirefoxBinary(firefox_path),
    firefox_profile=firefox_profile)
```

To this:

```python
driver = Firefox(
    capabilities=capabilities,
    executable_path=driver_path,
    firefox_options=firefox_options)
```

Since earlier this year, the python bindings have included a `firefox_options`
keyword argument, which provides a way to pass these (and more) options in a
more extensible way. This means there are now three ways to specify a binary
path and profile:

1. capabilities
1. `firefox_binary` and `firefox_profile`
1. `firefox_options`

Providing so many ways makes it difficult to cover all the edge cases. For
example, what should happen if a profile is specified in all three? To resolve
this, there's a good chance that the `firefox_binary` and `firefox_profile`
keyword arguments will be deprecated at some point in Selenium 3, and removed in
a future version.

I've decided therefore to stop using these keyword arguments in pytest-selenium,
which was achieved by simply switching to the new `firefox_options` keyword
argument. I'm not expecting users to notice any changes, except perhaps that we
now depend on a more recent version of the python bindings to ensure the new
keyword argument is available.

Along with this change, I have introduced a `firefox_options` fixture, which
allows users to replace or modify the options for specific tests. Here's an
example, which adds the `-foreground` argument to the options. Note that adding
Firefox arguments requires you to be using a recent Firefox and GeckoDriver.

```python
@pytest.fixture
def firefox_options(request, firefox_options):
    firefox_options.add_argument('-foreground')
    return firefox_options
```

You could even add a custom marker to specify arguments:

```python
@pytest.fixture
def firefox_options(request, firefox_options):
    args = request.node.get_marker('firefox_arguments')
    if args is not None:
        for arg in args.args:
            firefox_options.add_argument(arg)
    return firefox_options

@pytest.mark.firefox_arguments('-foreground')
def test_firefox_in_foreground(base_url, selenium):
    # your test code here
```

I considered adding this custom marker to pytest-selenium, but as with most
features I like to see them graduate from a test suite once they've proven
themselves useful, to avoid bloating the plugin.

For now, the command line options for specifying the Firefox path, profile,
extensions, and preferences will all still work. In the future I may remove
these in favour of using capabilities or the `firefox_options` fixture.
