---
layout: post
title: "The CI chain reaction that builds flickr downloadr"
date: 2014-12-08 14:01:26 -0500
description: As written on the last entry in this blog, I finally got to write something on points 1) and 2) there - the CI (continuous integration) builds that are running in parallel on AppVeyor, Travis CI and Wercker and the multi-platform installers they create and publish to the website. I wrote this up as documentation on the main GitHub repository but decided to cross-post here on the blog too.
comments: true
author: Floyd Pink
categories: [flickr downloadr, Technical, Windows, Mac, Linux, Continuous Integration, Travis CI, AppVeyor, Wercker]
---

>**Update (6/11/2015)** - Due to the recent news about malware on installers downloaded from SourceForge, I have decided to stay completely away from it. This is the reason for the move to GitHub releases for archiving the releases.
__________________

> As written on [the last entry](/blog/2014/06/12/batch-download-flickr-photos-from-windows-mac-or-linux/) in this blog, I finally got to write something on points 1) and 2) there - the CI (continuous integration) builds that are running in parallel on AppVeyor, Travis CI and Wercker and the multi-platform installers they create and publish to the website. I wrote this up as [documentation on the main GitHub repository](https://github.com/flickr-downloadr/flickr-downloadr-gtk/blob/bba17e54cdca04e07eb3422d07cc82888bdbb986/continuous-integration.md) but decided to cross-post here on the blog too.

### Automated Build Process

Every commit to the `master` branch [on the source code repository](https://github.com/flickr-downloadr/flickr-downloadr-gtk/) kicks off CI builds (that builds and runs all the unit tests) in three different CI services:

 - [AppVeyor](https://ci.appveyor.com/project/floydpink/flickr-downloadr-gtk) for Windows
 - [Travis CI](https://travis-ci.org/flickr-downloadr/flickr-downloadr-gtk) for Mac OS X
 - [Wercker](https://app.wercker.com/project/bykey/065aabc1580cec6d31a2daeef61548b0) for Linux

Any commit that has the string `[deploy]` in the commit message will also build the installers for all three supported platforms, using [BitRock InstallBuilder](http://installbuilder.bitrock.com/).

The installers will be committed separately from the CI builds to a new branch named `deploy-v<VERSION>` (as an example, for the version`v1.0.2.1`, the new branch will be `deploy-v1.0.2.1`) on the [`flickr-downloadr/flickr-downloadr.github.io`](https://github.com/flickr-downloadr/flickr-downloadr.github.io) repository.

[A custom webhook](https://github.com/flickr-downloadr/github-webhook) on the `flickr-downloadr/flickr-downloadr.github.io` repo that runs on Heroku ensures that installers for all three platforms have been built successfully and then makes a commit with the name of the new branch, updated into the `branch` file on the [`flickr-downloadr/releases`](https://github.com/flickr-downloadr/releases) repository (`deploy-v1.0.2.1`, in the example above) and sends an email a deployment will be ready to be pushed to the website soon [[see here](https://github.com/flickr-downloadr/github-webhook/blob/c88f106965878d62992db286fcdbca02385def1a/deploy/index.js#L59)].

In the event that any of the CI builds fail to create the installer, after at least one installer gets successfully built and committed, the custom webhook waits for 30 minutes from the first installer getting committed and sends an email notifying of a build failure [[see here](https://github.com/flickr-downloadr/github-webhook/blob/c88f106965878d62992db286fcdbca02385def1a/helpers/index.js#L68)].

Yet another webhook on the `flickr-downloadr/releases` repo kicks off another build on [Wercker](https://app.wercker.com/project/bykey/d981bd85d611e5bb2082c94959272851) to merge the new branch with all three installers (`deploy-v1.0.2.1` branch) into the `source` branch on `flickr-downloadr/flickr-downloadr.github.io` to make it ready for push into the `master` branch that runs [the main website](https://flickrdownloadr.com). This CI job does a few other things like archiving this version to <del>SourceForge</del>[GitHub releases](https://github.com/flickr-downloadr/flickr-downloadr-gtk/releases) etc., as can be seen [here](https://github.com/flickr-downloadr/releases/blob/master/wercker.yml).

The final push to the website is manual and can be done by running `grunt deploy` on the latest, merged version of the `source` branch and this will make the latest version installers current on the website.
