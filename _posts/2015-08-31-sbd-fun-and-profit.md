---
layout: post
date: 2015-08-31 16:51
title: Fencing for Fun and Profit with SBD
published: true
excerpt: SBD can be particularly useful in environments where traditional fencing mechanisms are not possible.
category:
tags: 
- cluster
- fencing
- concepts
---

## What is this Fencing Thing and do I Really Need it?

Fundamentally fencing is a mechanism for turning a question 

> Is node X capable of causing corruption?

into an answer

> No

so that the cluster can safely initiate recovery after a failure.

This question exists because we cannot assume that an unreachable node
is in fact off.

Sometimes it will do this by powering the node off, clearly a dead
node can do no harm.  Other times we will use a combination of network
(stop new work from arriving) and disk (stop a rogue process from
writing anything to shared storage) fencing.

Fencing is a requirement of _almost any_ cluster, regardless of
whether it is active/active, active/passive or involves shared storage
(or not).

One of the best ways of implementing fencing is with a remotely
accessible power switch, however some environments may not allow them,
see the value in them, or have ones that are suitable for clustering
(such as IPMI devices that loose power with the host they control).

## Enter SBD 

SBD can be particularly useful in environments where traditional
fencing mechanisms are not possible.

SBD integrates with Pacemaker, a watchdog device and, optionally,
shared storage to arrange for nodes to reliably self-terminate when
fencing is required (such as node failure or loss of quorum).

This is achieved through a watchdog device, which will reset the
machine if SBD does not poke it on a regular basis or if SBD closes
its connection "ungracefully".

Without shared storage, SBD will arrange for the watchdog to expire if:

- the local node looses quorum, or
- the Pacemaker, Corosync or SBD daemons are lost on the local node and are not recovered, or
- Pacemaker determines that the local node requires fencing, or
- in the extreme case that Pacemaker kills the sbd daemon as part of recovery escalation

When shared storage is available, SBD can also be used to trigger
fencing of its peers.

It does this through the exchange of messages via shared block storage
such as a SAN, iSCSI, FCoE.  SBD on the target peer sees the message
and triggers the watchdog to reset the local node.

These properties of SBD also make it particularly useful for dealing
with network outages, potentially between different datacenters, or
when the cluster needs to forcefully recover a resource that refuses
to stop.

Documentation is another area where diskless SBD shines, because
it requires no special knowledge of the user's environment.

## Not a Silver Bullet

One of the ways in which SBD recognises that the node has become
unhealthy is to look for quorum being lost.  However traditional
quorum makes no sense in a two-node cluster and is often disabled by
setting `no-qorum-policy=ignore`.

SBD will honour this setting though, so in the event of a network
failure in a two-node cluster, the node isn't going to self-terminate.

Likewise if you enabled Corosync 2's `two_node` option, both sides
will always have quorum and neither party will self-terminate.

It is therefor suggested to have three or more nodes when using SBD
without shared storage.

Additionally, using SBD for fencing relies on at least part of a
system that has already showed itself to be malfunctioning (otherwise
we wouldn't be fencing it) to function correctly.

Everything has been done to keep SBD as small, simple and reliable as
possible, however all software has bugs and you should choose an
appropriate level of paranoia for your circumstances.

### Installation

RHEL 7 and derivatives like CentOS include sbd, so all you need is `yum install -y sbd`.

For other distributions, you'll need to build it from source.

    # git clone git@github.com:ClusterLabs/sbd.git
    # cd sbd
    # autoreconf -i
    # ./configure

then either

    # make rpm

or

    # sudo make all install
    # sudo install -D -m 0644 src/sbd.service /usr/lib/systemd/system/sbd.service
    # sudo install -m 644 src/sbd.sysconfig /etc/sysconfig/sbd

> **NOTE:** The instructions here do not apply to the version of SBD that
> currently ships with openSUSE and SLES.

### Configuration

SBD's configuration lives in `/etc/sysconfig/sbd` by default and the
we include a sample to get you started.

For our purposes here, we can ignore the shared disk functionality and
concentrate on how SBD can help us recover from loss of quorum as well
as daemon and resource-level failures.

Most of the defaults will be fine, and really all you need to do is
specify the watchdog device present on your machine.

Simply set `SBD_WATCHDOG_DEV` to the path where we can find your
device and thats it.  Below is the config from my cluster:
 
    # grep -v \# /etc/sysconfig/sbd | sort | uniq
    SBD_DELAY_START=no
    SBD_PACEMAKER=yes
    SBD_STARTMODE=clean
    SBD_WATCHDOG_DEV=/dev/watchdog
    SBD_WATCHDOG_TIMEOUT=5

> **Beware**: If `uname -n` does not match the name of the node in the
> cluster configuration, you will need to pass the advertised name to
> SBD with the `-n` option. Eg. `SBD_OPTS="-n special-name-1"`

#### Adding a Watchdog to a Virtual Machine

Anyone experimenting with virtual machines can add a watchdog device
to an existing instance by editing the xml and restarting the
instance:

    virsh edit vmnode

Add `<watchdog model='i6300esb'/>` underneath the '<devices>'
tag. Save and close, then reboot the instance to have the config
change take effect:

    virsh destroy vmnode
    virsh start vmnode

You can then confirm the watchdog was added:

    virsh dumpxml vmnode | grep -A 1 watchdog 

The output should look something like:

    <watchdog model='i6300esb' action='reset'>
      <alias name='watchdog0'/>
      <address type='pci' domain='0x0000' bus='0x00' slot='0x02' function='0x0'/>
    </watchdog>

#### Using a Software Watchdog

If you do not have a real watchdog device, you should go out and get one.

However you're probably investigating SBD because it was not
possible/permitted to get a real fencing device, so there is a strong
chance you're going to using a software based watchdog device.

Software based watchdog devices are not evil incarnate however you
should be aware of their limitations, they are after-all software and
require a degree of correctness from a system that has already showed
itself to not be (functioning correctly, otherwise we wouldn't be
fencing it).

That being said, it still provides value when there is a network
outage, potentially between different datacenters, or the cluster
needs to forcefully recover a resource that refuses to stop.

To use a software watchdog, you'll need to load the kernel's `softdog` module:

    /sbin/modprobe softdog

Once loaded you'll see the device appear and you can set
`SBD_WATCHDOG_DEV` accordingly:

    # ls -al /dev/watchdog
    crw-rw----. 1 root root 10, 130 Aug 31 14:19 /dev/watchdog

Don't forget to arrange for the `softdog` module to be loaded at boot
time too:

    # echo softdog > /etc/modules-load.d/softdog.conf 

### Using SBD

On a `systemd` based system, enabling SBD with `systemctl enable sbd`
will ensure that SBD is automatically started and stopped whenever
`corosync` is.

If you're integrating SBD with a distro that doesn't support systemd,
you'll likely want to edit the `corosync` or `cman` init script to
both source the sysconfig file and start the `sbd` daemon.

#### Simulating a Failure

To see SBD in action, you could:

- stop pacemaker without stopping corosync, and/or
- kill the sbd daemon, and/or
- use `stonith_admin -F`

Killing `pacemakerd` is usually not enough to trigger fencing because
systemd will restart it "too" quickly.  Likewise, killing one of the
child daemons will only result in `pacemakerd` respawning them.

### Uninstalling

On every host, run:

    # systemctl disable sbd

Then on one node, run:

    # pcs property set stonith-watchdog-timeout=0
    # pcs cluster stop --all

At this point no part of the cluster, including Corosync, Pacemaker or
SBD should be running on any node.

Now you can start the cluster again and completely remove the
`stonith-watchdog-timeout` option:

    # pcs cluster start --all
    # pcs property unset stonith-watchdog-timeout

### Troubleshooting

SBD will refuse to start if the configured watchdog device does not exist.
You might see something like this:

    # systemctl status sbd
    sbd.service - Shared-storage based fencing daemon
       Loaded: loaded (/usr/lib/systemd/system/sbd.service; disabled)
       Active: inactive (dead)

To obtain more logging from SBD, pass additional `-V` options to the
`sbd` daemon when launching it.

SBD will trigger the watchdog (and your node will reboot) if `uname
-n` is different to the name of the node in the cluster configuration.
If this is the case for you, pass the correct name to `sbd` with the
`-n` option.

Pacemaker will refuse to start if it detects that SBD should be in use
but cannot find the `sbd` process. 

The `have-watchdog` property will indicate if Pacemaker considers SBD
to be in use:

    # pcs property 
    Cluster Properties:
     cluster-infrastructure: corosync
     cluster-name: STSRHTS2609
     dc-version: 1.1.12-a14efad
     have-watchdog: false
     no-quorum-policy: freeze

