---

title: ! 'Pacemaker 1.0.5: Testing In Progress'
tags: 
---
A quick note to say that 1.0.5 testing officially started today. Release
testing usually takes 1-2 weeks.

Currently queued changes for this release:

  * High (bnc#507255): Tools: crm: implement date expressions
  * High: Build: Fix compilation when snmp and esmtp are not available
  * High: Core: Show help text and exit with rc 1 if option processing failed
  * High: PE: Bug 2160 - Dont shuffle clones due to colocation
  * High: PE: Bug bnc#515172 - Correctly process location constraint rules which contain multiple expressions
  * High: PE: Bug bnc#515172 - Fix the boolean-op attribute of rules
  * High: PE: Fix reload for master/slave resources
  * High: PE: New implementation of the resource migration (not stop/start) logic
  * High: PE: Only prevent migration if the clone dependancy is stopping/starting on the target node
  * High: Tools: Differentiate between --help and an unknown option
  * High: Tools: crm: new display type (uppercase keywords)
  * High: Tools: crm: support for color output
  * High: ais: Fix cluster connection when using corosync 1.0
  * High: crmd: Terminate if we are ever evicted from the membership
  * High: crmd: Unset any existing DC value before querying for a new one
  * High: lrm: Look in the correct location for stonith agents

  * Medium: Extra: Add tools, an RA and tests for the System Health feature written by Mark Hamzy
  * Medium: PE: Prevent use-of-NULL in find_first_action()
  * Medium: Tools: crm_resource - Prevent use-of-NULL by requiring a resource name for the -A and -a options
  * Medium: cib: Supply an empty status section for replace operations

