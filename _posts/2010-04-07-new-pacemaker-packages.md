---
layout: post
title: New Pacemaker Packages
tags: 
---
I've begun uploading 1.0.8-3 to the clusterlabs.org servers.

Upon closer inspection, it became apparent that the 1.0.8-2 packages were
built with the wrong tarball and this led to some substantial problems with
the shell.

To rectify this, I've built 1.0.8-3. This new version uses the original 1.0.8
tarball and an updated spec file (to fix the snmp dependancies).

Apologies for the inconvenience.

