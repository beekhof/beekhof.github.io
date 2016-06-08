---
layout: post
date: 2014-11-14 21:17
title: Feature Spotlight - Smart Resource Restart from the Command Line
excerpt: Anatomy of resource restart
tag: Feature Spotlight
---

Restarting a resource can be a complex affair if there are things that
depend on that resource or if any of the operations might take a long
time.

Stopping a resource is easy, but it can be hard for scripts to
determine at what point the the target resource has stopped (in order
to know when to re-enable it), at what point it is appropriate to give
up, and even what resources might have prevented the stop or start
phase from completing.

For this reason, I am pleased to report that we will be introducing a
`--restart` option for `crm_resource` in Pacemaker 1.1.13.

## How it works

Assuming the following invocation

    crm_resource --restart --resource dummy

The tool will:

1. Check the current state of the cluster
1. Set the `target-role` for `dummy` to `stopped`
1. Calculate the future state of the cluster
1. Compare the current state to the future state
1. Work out the list of resources that still need to stop
1. If there are resources to be stopped 
   1. Work out the longest timeout of all stopping resource
   1. Look for changes until the timeout
   1. If nothing changed, indicate which resources failed to stop and exit
   1. Go back to step 4. 
1. Now that everything has stopped, remove the `target-role` setting for `dummy` to allow it to start again
1. Calculate the future state of the cluster
1. Compare the current state to the future state
1. Work out the list of resources that still need to stop
1. If there are resources to be stopped 
   1. Work out the longest timeout of all stopping resource
   1. Look for changes until the timeout
   1. If nothing changed, indicate which resources failed to start and exit
   1. Go back to step 9. 
1. Done

## Considering Clones

`crm_resource` is also smart enough to restart clone instances running
on specific nodes with the optional `--node hostname` argument.  In
this scenario instead of setting `target-role` (which would take down
the entire clone), we use the same logic as `crm_resource --ban` and
`crm_resource --clear` to enable/disable the clone from running on
the named host.

## Want to know more?

Drop by [IRC](irc://freenode.org#linux) or ask us a question on the [Pacemaker mailing list](http://clusterlabs.org/wiki/Mailing_lists)

There is also plenty of [documentation](http://clusterlabs.org/doc) available.
