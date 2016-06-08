---

title: Introducing the Pacemaker Master Control Process for Corosync-based Clusters
tags: 
---
The latest addition to the Pacemaker 1.1 series is a master control process
(MCP) and associated init script.

This means that Pacemaker is now started/stopped independently of the
messaging layer. We anticipate that this should result in a simpler and more
reliable startup/shutdown procedure when used in combination with Corosync.

> Forking inside a multi-threaded process like Corosync causes all sorts of
pain. This has been problematic for Pacemaker as it needs a number of daemons
to be spawned.

>

> Likewise, Corosync was never designed for staggered shutdown - something
previously needed in order to prevent the cluster from leaving before
Pacemaker could stop all active resources.

>

> By moving this functionality into the MCP, the whole system should become
more reliable

It should be noted that when using the MCP, **Corosync will refuse to shutdown
if Pacemaker is still running**. Pacemaker will also naturally fail to start
if Corosync isn't active yet.

So, starting with 1.1.3, the following Corosync-based options are possible:

  1. corosync + pacemaker plugin (v0) 
  2. corosync + pacemaker plugin (v1) + mcp
  3. corosync + cpg + cman + mcp
  4. corosync + cpg + quorumd + mcp

Option '1' corresponds to what people have been using since openais/corosync
started being supported. If Pacemaker starts being supported in RHEL6, its
probably going to look like option '3'. Option '4' is what we're all working
towards.

Anyone having startup or shutdown problems (with Pacemaker 1.1 or 1.0) should
immediately move to clusters based on option '2' or '3'.

Both involve the new master control process and therefor benefit from the more
reliable startup/shutdown design.

Additionally, '3' uses CPG for messaging (whereas '2' still uses the plugin
which makes it compatible with option nodes running '1').

Unfortunately option '4' isn't fully baked yet, there's still a few kinks in
the pacemaker/quorumd interaction to be worked out. This will happen in the
coming months, however any assistance in this process would be highly
appreciated.

To use option '2', simply change: `ver: 0` to `ver: 1` in the pacemaker
service block of _corosync.conf_.

To use option '3', you can either: * use _cluster.conf_ and `service cman
start` or, * add the cman bits to _corosync.conf_.

Using _cluster.conf_ is the preferred approach. Its far easier to maintain and
start automatically starts the necessary pieces for using GFS2.

### Alternative 1 - Sample cluster.conf for a two-node cluster

    
    <?xml version="1.0"?>
    <cluster config_version="1" name="beekhof">
        <fence_daemon clean_start="0" post_fail_delay="0" post_join_delay="3"/>
        <clusternodes>
                <clusternode name="pcmk-1" nodeid="1" votes="1">
                        <fence/>
                </clusternode>
                <clusternode name="pcmk-2" nodeid="2" votes="1">
                        <fence/>
                </clusternode>
        </clusternodes>
        <cman/>
        <fencedevices/>
        <rm/>
    </cluster>
    

### Alternative 2 - Sample corosync.conf additions for a two node CMAN cluster

**Be sure to set `nodename` appropriately for each host.**
    
    cluster {
        name: beekhof
    
        clusternodes {
                clusternode {
                        votes: 1
                        nodeid: 1
                        name: pcmk-1
                }
                clusternode {
                        votes: 1
                        nodeid: 2
                        name: pcmk-2
                }
        }
        cman {
                expected_votes: 2
                cluster_id: 123
                nodename: pcmk-1
                two_node: 1
                max_queued: 10
        }
    }
    
    service {
        name: corosync_cman
        ver: 0
    }
    
    quorum {
        provider: quorum_cman
    }
    

