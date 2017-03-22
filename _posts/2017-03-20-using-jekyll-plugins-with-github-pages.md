---
layout: post
title: Using Jekyll plugins with GitHub pages
author: Dave Hunt
tags: [jekyll, github pages, plugins]
comments: true
---
I was a little disappointed to discover that Jekyll doesn't offer a way to view
all posts with a particular tag. Fortunately there's a comprehensive library of
plugins, including [jekyll-tagging](https://github.com/pattex/jekyll-tagging),
however as I was letting [GitHub pages](https://pages.github.com/) take care of
building my site, I was limited to the
[list of supported plugins](https://pages.github.com/versions/). To work around
this, I moved from a User Pages site to a Project Page site so that I could use
separate branches for the source and generated content (User Pages only allows
the `master` branch to be used). Then, I found the neat
[jekyll-github-deploy](https://github.com/yegor256/jekyll-github-deploy) tool,
which builds the site and pushes to the `gh-pages` branch. So now each post
includes a list of tags, and each tag links to a page listing the posts
associated with that tag.<!--more-->

In making this change I've decided to drop the categories that were previously
associated with posts, as I believe the tags are much more useful.
Unfortunately, this means that my permalinks have changed, but seeing as this
blog is less than a year old and has less that ten posts I'm not too concerned.
