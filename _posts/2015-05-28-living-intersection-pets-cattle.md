---

date: 2015-05-28 12:50
title: Life at the Intersection of Pets and Cattle
published: true
excerpt:
 The theory goes that pets have no place in the server room.
 Everything should be stateless and replicated.
 If one copy dies, who cares, there are 100 more.
category:
tags:
- openstack

---

>
> Scale UP: Servers are like pets - you name them and when they get sick you nurse them back to health
>
> Scale OUT: Servers are like cattle - you number them and when they get sick you shoot them
>

### Why Pets?

The theory goes that pets have no place in the server room.
Everything should be stateless and replicated and if one copy dies, who cares because there are 100 more.

Except real life isn't like that.

It's hard to build replicated, stateless, shared-nothing systems, its
even hard just bringing them online because the applications often
need external context in order to distinguish between different
recovery scenarios.

I cover some of these ideas in my [Highly Available Openstack
Deployments](https://github.com/beekhof/osp-ha-deploy/blob/master/ha-openstack.md#cluster-manager)
document.

Indeed that document shows that even the systems built to manage
cattle contain pieces that need to be treated as pets.  They
demonstrate the very limitations of the model they advocate.

### Life at the Intersection

Eventually someone realises they need a pet after-all.

This is when things get interesting, because baked in from the start
is the assumption that we don't need to care about cattle:

- It doesn't matter if some of the cattle die, there's plenty more
- It doesn't matter if some of the cattle die when a paddock explodes, there's plenty more
- It doesn't matter if some of the cattle are lost moving them from one paddock to another, there's plenty more
- It doesn't matter if new cattle stock is lost unloading them into a new paddock, there's plenty more
- ...
- It doesn't matter, just try again

The assumptions manifest themselves in a variety of ways:

- Failed actions are not retried
- Error reporting is confused for error handling
- Incomplete records (since the cattle can be easily re-counted)

All of which makes adopting some cattle as pets _really_ freaking hard.

### Raising Pets in Openstack

Some things are easier said than done.

> When the compute node hosting an instance dies, evacuate it elsewhere

Easy right?

All we need to do is notice that the compute node disappeared, make sure its _really_ dead (otherwise it might be running twice, which would be bad), and pick someone to call evacuate.

Except:

- You can't call evacuate before nova notices it's peer is gone
- You can't (yet) tell nova that it's peer has gone

Ok, so we can define a fencing device that talks to nova, loops until it notices the peer is gone and calls evacuate.

Not so fast, the failure that took out the compute node may have also taken out part of the control plane, which needs fencing to complete before it can be recovered.
However in order for fencing to complete, the control plane needs to have recovered (nova isn't going to be able to tell you it noticed the peer died if your request can't be authenticated).

Ok, so we can't use a fencing device, but the cluster will let services know when their peers go away.
The notifications are even independent of the recovering the VIPs, so as long as at least one of the control nodes survives, we can block waiting for nova and make it work. 
We just need to arrange for only one of the survivors to perform the evacuations.

Job done, retire...

Not so fast kimosabi.
Although we can recover a single set of failed compute and/or control nodes, what if there is a subsequent failure?
You've had one failure, that means more work, more work means more opportunities to create more failures.

Oh, and by the way, you can't call evacuate more than once.
Nor is there a definitive way to determine if an instance is being evacuated.

Here are some of the ways we could still fail to recover instances:

- A compute node that is in the process of initiating evacuations dies  
  It takes time for nova to accept the evacuation calls, there is a window for some to be lost if this node dies too.
- A compute node which is receiving an evacuated node dies  
  At what point does the new compute node "own" the instance such that if this node died to, the instance would be picked up by a subsequent evacuate call?
  Depending on what is recorded inside nova and when, you might have a problem.
- The control node which is orchestrating an evacuation dies  
  There a window between a request being removed from the queue and it being actioned to the point that it will complete?
  Depending on what is recorded inside nova and when, you might have a problem.
- A control node hosting one of the VIPs dies while an evacuation is in progress (probably)  
  Do any of the activities associated with evacuation require the use of inter-component APIs?
  If so, you might have a problem if one of those APIs is temporarily unavailable 
- Some other entity (human or otherwise) also initiates an evacuation
  If there is no way for nova to say if an instance is being evacuated, how can we expect an admin to know that initiating one would be unsafe?
- It is the 3rd Tuesday of a month with 5 Saturdays and the moon is a waxing gibbous  
  Ok, perhaps I'm a little jaded at this point.

### Doing it Right

Hopefully I've demonstrated the difficulty of adding pets (highly available instances) as an after thought.
All it took to derail the efforts here was the seemingly innocuous decision that the admin should be responsible for retrying failed evacuations (based on it not having appeared somewhere else after a while?).
Who knows what similar assumptions are still lurking.

At this point, people are probably expecting that I put my Pacemaker hat on and advocate for it to be given responsibility for all the pets.
Sure we could do it, we could use nova APIs to managed them just like we do when people use their hypervisors directly.

But that's never going to happen, so lets look at the alternatives.
I foresee three main options:

1. First class support for pets in nova  
   Seriously, the scheduler is the best place for all this, it has all the info to make decisions and the ability to make them happen.

2. First class support for pets in something that replaces nova  
   If the technical debt or political situation is such that nova cannot move in this direction, perhaps someone else might.

3. Creation of a distributed finite state machine that:  
   - watches or is somehow told of new instances to track
   - watches for successful fencing events
   - initiates and tracks evacuations
   - keeps track of its peer processes so that instances are still evacuated in the event of process or node failure

The cluster community has pretty much all the tech needed for the last option, but it is mostly written in C so I expect that someone will replicate it all in Python or Go :-)

If anyone is interested in pursuing capabilities in this area and would like to benefit from the knowledge that comes with 14 years experience writing cluster managers, drop me a line.
