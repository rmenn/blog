+++
date = "2017-03-24T21:31:51+05:30"
title = "Bootkube"
tags = ["bootkube","kubernetes"]
categories = ["bootkube"]
draft = false
description = "Self-hosted kubernetes"

+++
I have been trying out bootkube for a while now. It may be a beta, but I love the concept and how it works. 

For those who have not heard about [bootkube][1], its a tool which is used
to bootstrap self-hosted clusters. Self-hosted essentially means running kubernetes
as a objects on the kubernetes cluster ( I can hear people close their tabs ), its just
kubernetes running on kubernetes. You can take a look at this video where Aaron Levy explains
what self-hosted is and how you can benefit from it.
{{< youtube EbNxGK9MwN4 >}}

The one thing that you need to know is that bootkube is very opinionated on how the cluster
should be. Bootkube is nothing but a tool to bootstrap your own self-hosted cluster.

bootkube helps me where i need it the most, upgrading clusters, i know upgrading to 1.6 is not going to be trivial, but i have done
multiple upgrades 1.5.1 all the way to 1.5.4. 

[1]: https://github.com/kubernetes-incubator/bootkube
