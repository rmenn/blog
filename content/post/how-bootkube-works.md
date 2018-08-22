+++
title = "How bootkube works"
date = "2018-08-21"
+++

 What is bootkube

bootkube is a project in the kubernetes incubator which allows you to self host the control-plane of kubernetes. Simple, isnt it?

What is self hosting

The idea of self hosting is to run the kubernetes control-plane in kubernetes leveraging kubernetes objects. That is, to run the api-server, controller-manager & the scheduler as kubernetes objects, which then enables you to manage kubernetes like any other application you run on kubernetes. What does this mean? You manage kubernetes componets using your favorite tool `kubectl`. 
