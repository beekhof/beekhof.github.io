---
date: 2018-02-16 14:32
title: A New Thing
excerpt: I made something new, maybe you'll find it useful
tags:
- high availability
- kubernetes
- galera
---

I made a [new thing](https://github.com/beekhof/rss-operator).

If you're interested in [Kubernetes](https://kubernetes.io) and/or managing
replicated applications, such as Galera, then you might also be interested in
an [operator](https://coreos.com/blog/introducing-operators.html) that allows
this class of applications to be managed natively by Kubernetes.

There is plenty to read on [why](https://github.com/beekhof/rss-operator/blob/master/doc/Rationale.md)
the operator exists, [how](https://github.com/beekhof/rss-operator/blob/master/doc/design/replication.md)
replication is managed and the steps to [install it](https://github.com/beekhof/rss-operator/blob/master/doc/user/install_guide.md)
if you're interested in trying it out.

There is also a screencast that demonstrates the major concepts:

[![asciicast](https://asciinema.org/a/164903.png)](https://asciinema.org/a/164903)

Feedback welcome.
