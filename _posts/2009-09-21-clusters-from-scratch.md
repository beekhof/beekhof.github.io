---

title: Clusters From Scratch
tags:
- announce
- documentation
---
The first of a new series of step-by-step guides for Pacemaker.

This installment covers installation, the creation of an
[active/passive](http://en.wikipedia.org/wiki/High-availability_cluster#Node_configurations)
cluster and its conversion to active/active.

Technologies used include:

  * Fedora 11 as the host operating system
  * OpenAIS to provide messaging and membership services,
  * Pacemaker to perform resource management,
  * DRBD as a cost-effective alternative to shared storage,
  * OCFS2 as the cluster filesystem (in active/active mode)
  * The crm shell for displaying the configuration and making changes
  * Apache as the example service. 

The PDF is available from our [Documentation page](http://www.clusterlabs.org/wiki/Documentation#Howtos) or directly via
[http://www.clusterlabs.org/mediawiki/images/9/9d/Clusters_from_Scratch_-_Apache_on_Fedora11.pdf](http://www.clusterlabs.org/mediawiki/images/9/9d/Clusters_from_Scratch_-_Apache_on_Fedora11.pdf)

Future guides are anticipated to include MySQL, mail servers and asymmetrical
clusters. Feedback and suggestions for additional topics are welcome.

