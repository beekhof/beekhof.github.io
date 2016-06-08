---
layout: post
title: "Pacemaker on RHEL6.4"
date: 2013-05-04 8:09
comments: true
tags:
---

Over the last couple of years, we have been evolving the stack in two
ways of particular relevance to RHEL customers:

 - minimizing the differences to the default RHEL-6 stack to reduce
   the implication of supporting Pacemaker there (making it more
   likely to happen)

 - adapting to changes to Corosync's direction (ie. the removal of
   plugins and the addition of a quorum API) for the future

As a general rule, Red Hat does not ship packages it doesn't at least
plan to support.  So part of readying Pacemaker for "supported" status
is removing or deprecating the parts of Red Hat's packages that they
have no interest and/or capacity to support.

### Removal of crmsh

For reasons that you may or may not agree with, Red Hat has decided to
rely on `pcs` for command line and GUI cluster management in RHEL-7.

As a result there is no future, in RHEL, for the original cluster
shell `crmsh`.

Normally it would have been deprecated.  However since `crmsh` is now
a stand-alone project, it's removal from the Pacemaker codebase also
resulted in it's removal from RHEL-6 once the packages were refreshed.

To fill the void and help prepare people for RHEL-7, `pcs` is now also
available on RHEL-6.

### Status of the Plugin

Anyone taking the recommended approach of using Pacemaker with CMAN
(ie. `cluster.conf`) on RHEL-6 or any of its derivatives can stop
reading for now (we'll need to talk again when RHEL 7 comes out with
corosync-2, but thats another conversation).

> Anyone using `corosync.conf` on RHEL 6 should keep reading...

One of the differences between the Pacemaker and rgmanager stacks is
where membership and quorum come from.

Pacemaker has traditionally obtained it from a custom plugin, whereas
rgmanager used CMAN.  Neither source is "better" than the other, the
only thing that matters is that [everyone obtains it from the same place](/blog/2012/pacemaker-and-cluster-filesystems/).

Since the rest of the components in a RHEL-6 cluster use CMAN, support
for it was added to Pacemaker which also helps minimize the support
load.  Additionally, in RHEL-7, Corosync's support for plugins such as
Pacemaker's (and CMAN's) goes away.

Without any chance of being supported in the short or long-term,
configuring plugin-based clusters (ie. via `corosync.conf` ) is now
officially deprecated in RHEL.  As some of you may have already
noticed, starting `corosync` in 6.4 produces the following entries in
the logs:

    Apr 23 17:35:36 corosync [pcmk  ] ERROR: process_ais_conf: You have configured a cluster using the Pacemaker plugin for Corosync. The plugin is not supported in this environment and will be removed very soon.
    Apr 23 17:35:36 corosync [pcmk  ] ERROR: process_ais_conf:  Please see Chapter 8 of 'Clusters from Scratch' (http://clusterlabs.org/doc) for details on using Pacemaker with CMAN

> **Everyone** is highly encouraged to switch to CMAN-based Pacemaker clusters

While the plugin will still be around, running Pacemaker in
configurations that are not well tested by Red Hat (or, for the most
part, by upstream either) contains an element of risk.

For example, the messages above were originally added for 6.3, however
since logging from the plugin was broken for over a year, no-one
noticed. It only got fixed when I was trying to figure out why no-one
had complained about them yet!

A lack of logging is annoying but not usually problematic,
unfortunately there is also something far worse...

### Fencing Failures when using the Pacemaker plugin

It has come to light that fencing for plugin-based clusters is
critically broken.

The cause was a single stray 'n'-character, probably from a
copy+paste, that prevents the `crmd` from correctly reacting to a
membership-level failures (ie. `killall -9 corosync`) of it's peers.

The problem did not show up in any of Red Hat's testing because of the
way Pacemaker processes talk to their peers on other nodes when CMAN
(or Corosync 2.0) is in use.

For CMAN and Corosync 2.0 we use Corosync's CPG API which provides
notifications when peer *processes* (the `crmd` in this case) join or
leave the messaging group.  These additional notifications from CPG
follow a different code path and are unaffected by the bug... allowing
the cluster to function as intended.

Unfortunately, despite the size and obviousness of the fix, a z-stream
update for a bug affecting a deprecated use-case of an
as-yet-unsupported package is a total non-starter.

> People wanting to stick with plugin-based clusters should obtain
> 1.1.9 or later from the [Clusterlabs repos](http://clusterlabs.org/rpm-next)
> that includes the fix

You can read more about the bug and the fix on the [Red Hat bugzilla](https://bugzilla.redhat.com/show_bug.cgi?id=951340)

For details on converting to a CMAN-based stack, please see [Clusters from Scratch](http://clusterlabs.org/doc/en-US/Pacemaker/1.1-plugin/html/Clusters_from_Scratch/_adding_cman_support.html).

> Switching to CMAN is really far less painful than it sounds

There is also a [quickstart guide](http://clusterlabs.org/quickstart-redhat.html)
for easily generating `cluster.conf`, just substitute the name of your cluster
nodes.
