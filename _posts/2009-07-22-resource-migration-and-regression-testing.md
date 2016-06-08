---

title: Resource Migration and Regression Testing
tags:
- internals
---
Yesterday I was working on a migration bug.

It didn't take long to identify or fix, and afterwards I was terribly pleased
with myself.

The fix was simple, elegant and allowed the cluster to use migration (instead
of stop then start) more often.

Why had I not seen how easy it was sooner?

Unfortunately it was because I'd ignored half the problem.

One decision I'm particularly happy I made place 6 years ago, is the one when
I ensured Pacemaker's Policy Engine could be used outside of a running
cluster. Combined with some additional output options, this makes it possible
to have a suite (224 right at this moment) of regression tests that catch this
sort of idiocy before it ever affects an actual user.

The Policy Engine is by far the most complex part of the system, and it's
totally infeasible to test by hand even a small fraction what the regression
suite can (and in under 30s too!).

This is why I'm so confident when I say that each release is better than the
last. Once a Policy Engine bug gets fixed, it _stays_ fixed.

The benefits of offline testing also occur much earlier in the
process. The Policy Engine keeps a rolling list of the cluster states
it performed calculations on and our test reporting tool collects
these. So when users report a Policy Engine bug, there is no need to
reproduce the issue and afterwards we can conclusively show (using
[pretty dot graphs](http://www.graphviz.org/) of the cluster's old and
new behavior) that the issue is resolved.

So I sat down again today and made sure I'd thought the whole problem through
- so that the next version would be a complete solution. You can see my notes
below if you're interested.

And now I'm off to implement it (and some extra tests :-)

## Migration Scenario Notes

### Cluster Setup

primitive(A) depends on clone(B)

### Resource Activity During Move: A(node-1 to node-2)

<table><tr><th>time</th><th>node-1</th><th>node-2</th><th>node-3
</th></tr><tr><td>t0&nbsp;&nbsp;&nbsp;&nbsp;</td><td>A.stop</td>
</tr><tr></tr><td>t1</td><td>B.stop</td><td></td><td>B.stop</td>
<tr></tr><td>t2</td><td></td><td>B.start</td><td>B.start</td>
<tr></tr><td>t3</td><td></td><td>A.start</td><td> </td>
<tr></tr></table><br/>

### Resource Activity During Migration: A(node-1 to node-2)

<table><tr><th>time</th><th>node-1</th><th>node-2</th><th>node-3
</th></tr><tr><td>t0&nbsp;&nbsp;&nbsp;&nbsp;</td><td></td><td>B.start</td><td>B.start</td>
</tr><tr><td>t1</td><td>A.stop*</td>
</tr><tr></tr><tr><td>t2</td><td></td><td>A.start**</td><td> </td>
</tr><tr><td>t3</td><td>B.stop</td><td></td><td>B.stop</td> </tr></table><br/>

  * Node *: Rewritten to be a migrate-to operation
  * Node **: Rewritten to be a migrate-from operation

### Constraints

The following constraints already exist in the system. The 'ok' and 'fail'
column refers to whether they still hold for migration.

  1. A.stop -> A.start - ok
  2. B.stop -> B.start - fail
  3. A.stop -> B.stop - ok
  4. B.start -> A.start - ok
  5. B.stop -> A.start - fail
  6. A.stop -> B.start - fail

### Scenarios

  1. B unchanged - ok
  2. B stopping only - fail - possible after reversing constraint 5
  3. B starting only - fail - possible after reversing constraint 6***
  4. B stoping and starting - fail - constraint 2 is unfixable
  5. B restarting but only on N2 - fail - as per case 4 but even less likely 

Note ***: This is what the existing implementation does

