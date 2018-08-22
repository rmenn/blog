+++
title = "How bootkube works"
date = "2018-08-21"
+++

 What is bootkube

bootkube is a project in the kubernetes incubator, primarly developed by coreos, which allows you to self host the control-plane of kubernetes. Simple, isnt it?

What is self hosting

The idea of self hosting is to run the kubernetes control-plane in kubernetes leveraging kubernetes objects. That is, to run the api-server, controller-manager & the scheduler as kubernetes objects, which then enables you to manage kubernetes like any other application you run on kubernetes. What does this mean? You manage kubernetes componets using your favorite tool `kubectl`. 


Why Self host?

As mentioned above you can interact with kubernetes as it was any other application running on kubernetes, so updating kubernetes becomes as simple as `kuberctl set-image daemonset/kube-apiserver kube-apiserver=quay.io/coreos/hyperkube:v1.10.4_coreos.0`
