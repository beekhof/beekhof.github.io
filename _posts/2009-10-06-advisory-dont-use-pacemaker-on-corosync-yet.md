---

title: ! 'Advisory: Don''t use Pacemaker on Corosync (yet)'
tags:
- announce
---
I spent some time looking into the state of the Pacemaker/Corosync integration
today and I can only recommend Pacemaker users stay on the previous version of
OpenAIS (aka. Whitetank).

In a nutshell, shutdown is utterly broken.

r2140 of Corosync removed the shutdown worker thread which allowed plugins
such as Pacemaker to continue sending and receiving cluster messages.

Without it, Corosync waits for Pacemaker to finish and Pacemaker waits for the
messages it tried to send to arrive and be acted upon. Needless to say no-one
makes any progress.

Stay tuned, now that integration testing has started it shouldn't take too
long to get everything sorted out.

> ### Update
>
> Since writing this, the necessary testing has been done and
> Pacemaker is now supported on Corosync provided you have `corosync >=
> 1.1.2` and `pacemaker >= 1.0.6`

