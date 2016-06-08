---

title: Configuring Heartbeat v1 Was So Simple
tags:
- tips
---
…because it couldn't do anything.

People who loved how simple Heartbeat v1 was to configure often complain how
complex Pacemaker is.

But the key differences between the two configurations are driven by the very
features that `haresources`-based clusters couldn't provide.

Granted we made a mess of things with the original XML syntax.

    

When the job of writing the CRM/Pacemaker was first pitched to me, I was
promised an all-singing, all-dancing GUI that would hide the ugly XML from end
users.

> So we focused on power and expressiveness and little thought was
> given to it's usability.

However with the release of Pacemaker 1.0 in October 2008, not only has the
XML been cleaned up, but Dejan has written a consolidated cluster shell **that
hides all the XML** anyway.

As a result **almost everything you thought about Pacemaker's complexity is no
longer true** and it's just as suited to simple two-node setups as it is to
large active/active configurations.

## Establishing A Baseline

Here's a sample `haresources` file from the linux-ha.org website:

    
    linuxha1 IPaddr::192.168.85.3 httpd smb
    

The pattern here is:

    
    $preferred_node $script::$parameter $script $script
    

Which is mostly sufficient for a two node cluster, because anything not
running on $preferred_node is running on _the only other machine in the
cluster_.

## Deficiencies

Of course there were some obvious limitations built into Heartbeat v1 that
Pacemaker was designed to address.

  * Couldn't support more than two nodes
  * Couldn't detect or recover from resource-level failures
  * Limited to sets of resources with a strict linear stop/start order

## Power Creates Complexity, but Not That Much

### Non-linear Resource Model

Consider the following environment:

  * Resource B depends on Resource A
  * Resource C depends on Resource A
  * Resource B and Resource C are independent

There is no way to truly express this with `haresources`.

In order to have a more powerful resource model, you need to be able to refer
multiple times to resources. So now they need a name and a more flexible way
to group them.

#### New Resource Syntax

This is Pacemaker's XML equivalent of the IPaddr resource above, which I've
imaginatively called _IP_:

    
        <primitive class="ocf" id="IP" provider="heartbeat" type="IPaddr">
          <instance_attributes id="IP-instance_attributes">
            <nvpair id="IP-instance_attributes-ip" name="ip" value="192.168.85.3"/>
          </instance_attributes>
        </primitive>
    

This is also the point at which many people run away screaming. There's really
no need for this though, almost all of it is scaffolding.

Here's how I created the above XML:

    
    # crm configure primitive IP ocf:heartbeat:IPaddr params ip=192.168.85.3
    

Which is composed of

  * primitive ::= The type of resource object that we're creating.
  * IP ::= Our name for the resource
  * IPaddr ::= The script to call
  * ocf ::= The standard it conforms to
  * ip=192.168.85.3 ::= Parameter(s) as name/value pairs

To create the other two members of the group, I ran:

    
    # crm configure primitive http lsb::httpd
    # crm configure primitive samba lsb::smb
    

Admit it, that was pretty easy :-)

To group them together, as-per the `haresources` example, is also trivial.
Just provide a name for the group and a list of members:

    
    # crm configure group v1-group IP http samba
    

Thats it! Here's the result:

    
    # crm configure show
    primitive IP ocf:heartbeat:IPaddr params ip="192.168.85.3"
    primitive http lsb:httpd
    primitive samba lsb:smb 
    group v1-group IP http samba
    

### Resource Recovery

In order to detect resource failure, the cluster needs to check its health
periodically. But what action should it call and how often? There is nowhere
in `haresources` to specify this. Not cleanly anyway.

By way of example, here's what it looks like in Pacemaker if you want to
monitor the IP address every 5 minutes and apache/samba once a minute:

    
    # crm configure show
    primitive IP ocf:heartbeat:IPaddr params ip="192.168.85.3" op monitor interval=5min
    primitive http lsb:httpd op monitor interval=60s
    primitive samba lsb:smb op monitor interval=60s
    group v1-group IP http samba
    

### More Than Two Nodes

Supporting more than two nodes means that can no longer specify a preferred
node like in v1. Thats not enough to tell the cluster where to put the
resource after `$preferred_node` fails. Instead you need an ordered list.

Here's an example of how we might specify that we prefer linuxha1 over
linuxha2 over linuxha3:

    
    # crm configure location prefer-ha1 v1-group 5000: linuxha1
    # crm configure location prefer-ha2 v1-group 500:  linuxha2
    # crm configure location prefer-ha3 v1-group 50:   linuxha3
    

The numbers (5000, 500, 50) are scores that indicate a relative preference for
running on the three nodes.

To finish off, here's the Pacemaker equivalent of just the original
`haresources` example:

    
    primitive IP ocf:heartbeat:IPaddr params ip="192.168.85.3"
    primitive http lsb:httpd
    primitive samba lsb:smb
    
    group v1-group IP http samba
    location prefer-ha1 v1-group 5000: linuxha1
    

See, its really not that scary and, as we saw, easily extendable to a more
complex clustering environment if needed.

#### Failback? Sure, but to Where?

Imagine the scenario, `linuxha1` failed and the group moved to `linuxha2`.
Then `linuxha2` failed, and now the resource group is running on `linuxha3`.
What happens when the other two nodes recover?

In v1, this is controlled by the `auto_failback` directive. But when there are
more than two nodes, where does `back` refer to? It could mean `linuxha2`,
since that was the last place it was running. It could also mean `linuxha1`,
since that is the most preferred node under normal circumstances.

The sanest way to resolve this ambiguity is to invert the option and make it a
preference for keeping the resource where it is.

Thus the `resource-stickiness` option was born and, unlike `auto_failback`, it
can be specified per-resource (with a global default). Which makes sense,
since the cost of stopping and relocating an IP address is significantly less
than that of an Oracle database.

Pacemaker even allows the administrator to have different fail-back policies
apply during "core" and "non-core" hours. But more on that another day.

