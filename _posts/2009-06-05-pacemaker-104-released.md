---

title: Pacemaker 1.0.4 Released
tags:
- announce
---
It took a little longer than expected, but the latest 1.0 maintenance release
(1.0.4) is finally available.

Apart from a number of important bug fixes, the latest release is the first to
include comprehensive man-pages for all CLI tools. These are generated from
the source code using [help2man](http://www.gnu.org/software/help2man) and so
are guaranteed to be accurate.

Unfortunately for RHEL and CentOS users, those distros don't ship help2man and
so the man pages are not available on those platforms. However one can obtain
the same information using the `--help` option.

Packages for Pacemaker 1.0 and it's immediate dependancies can be downloaded
for openSUSE, SLES, Fedora, RHEL, CentOS and Mandriva (Debian users read on)
from the usual location: [http://software.opensuse.org/download/server:/ha-clustering](http://software.opensuse.org/download/server:/ha-clustering)

and the source can be obtained from: [http://hg.clusterlabs.org/pacemaker/stable-1.0/archive/Pacemaker-1.0.4.tar.bz2](http://hg.clusterlabs.org/pacemaker/stable-1.0/archive/Pacemaker-1.0.4.tar.bz2)

General installation instructions are available at:
[http://clusterlabs.org/wiki/Install](http://clusterlabs.org/wiki/Install)

### Release Statistics

## Details
<table><tr><td><strong>Changesets&nbsp;</strong></td> <td>222</td>
</tr><tr><td><strong>Diff</strong></td> <td>266 files changed, 12100 insertions(+), 8279 deletions(-)</td> </tr></table><br/>


### Project Administrivia

#### Next Release

Yours truly will be on vacation for the second half of this month, so the next
release will be in late July.

#### RHEL-4 Packages

The build issues for RHEL-4 have finally been sorted out and binary packages
are one again available from the [build service](http://download.opensuse.org/repositories/server:/ha-clustering/RHEL_4/).

#### NEW! "Real" Debian Packages

By happy co-incidence, courtesy of Martin Loschwitz from
[LINBIT](http://www.linbit.com/), the 1.0.4 release sees the arrival of a
fully functional, "official", repository from which to obtain Pacemaker.

Martin's work replaces the _sort-of-worked-sort-of-didnt_ packages from the
[openSUSE build service](http://build.opensuse.org/) which have now been
disabled.

Too install packages for Lenny or Sid (Etch should be available "soon") see
the following instructions:

  1. Add _**one**_ of the following lines to `/etc/apt/sources.list` ` deb [http://people.debian.org/~madkiss/ha](http://people.debian.org/~madkiss/ha) lenny main deb [http://people.debian.org/~madkiss/ha](http://people.debian.org/~madkiss/ha) sid main `
  2. Retrieve the package metadata with: `apt-get update`
  3. Install either the OpenAIS or Heartbeat version:

`apt-get install pacemaker-openais`

or

`apt-get install pacemaker-heartbeat`

### Changes of note

  * High: ais: bnc#488291 - don't rely on byte endianness on ptr cast
  * High: Tools: bnc#507255 - crm: import properly rsc/op_defaults
  * High: Tools: lf#2114 - crm: add support for operation instance attributes
  * High: ais: Bug lf#2126 - Messages replies cannot be routed to transient clients
  * High: attrd: Support the value++ and value+=… syntax required for failcounts
  * High: cib: Fix huge memory leak affecting heartbeat-based clusters
  * High: Core: Generate the help text directly from a tool options struct
  * High: crmd: Bug lf#2120 - All transient node attribute updates need to go via attrd
  * High: crmd: Fix another large memory leak affecting Heartbeat based clusters
  * High: PE: Bug bnc#495687 - Filesystem is not notified of successful STONITH under some conditions
  * High: PE: Make running a cluster with STONITH enabled but no STONITH resources an error and provide details on resolutions
  * High: PE: Prevent use-of-NULL when using resource ordering sets
  * High: Tools: attrd - Prevent race condition resulting in the cluster forgetting node's wish to shut down
  * High: Tools: crm_mon - Fix smtp notifications
  * High: Tools: crm_resource - Repair the ability to query meta attributes
  * Medium: Core: Include supported stacks in version information
  * Medium: Tools: Include current stack in crm_mon output
  * Medium: PE: Correctly log the actions for resources that are being recovered
  * Medium: PE: Correctly log the occurance of promotion events
