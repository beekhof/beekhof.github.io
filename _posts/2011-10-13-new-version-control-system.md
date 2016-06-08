---
layout: post
title: New Version Control System
tags: 
---
Since September, Pacemaker has started using [Git](http://git-scm.com/) for
the 1.1 and devel trees.

There were some minor technical advantages over
[Mercurial](http://mercurial.selenic.com/) (which I still personally prefer),
but mostly the decision was driven by the pain associated with switching
between SCMs multiple times a day.

The majority of development now happens on
[GitHub](https://github.com/ClusterLabs/pacemaker), which has some great
features for [reviewing patches](http://github.com/features/projects/codereview) and general
collaboration.

The Pacemaker tree is also periodically sync'd to the [ClusterLabs](http://git.clusterlabs.org/) 
server in case GitHub is unavailable for any reason.

For those new to Git, GitHub has many tips for 
[setting up](http://help.github.com/set-up-git-redirect) Git, creating a 
[local copy](http://help.github.com/fork-a-repo/) of the Pacemaker repo to work in,
[submitting your changes](http://help.github.com/send-pull-requests/) upstream
(we use the Fork + Pull Model), and other
[assorted resources](http://help.github.com/git-cheat-sheets/).

Be sure to configure 
[email and user](http://help.github.com/set-your-user-name-email-and-github-token/) 
information so you get credit for your hard work too!

