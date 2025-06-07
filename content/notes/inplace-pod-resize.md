+++
date = '2025-06-07T17:29:30+05:30'
draft = true
title = 'Inplace Pod Resizeing'
+++


##### In Place pod resizing

In place pod resizing comes to beta with kubernetes 1.33. [Docs](https://github.com/kubernetes/enhancements/tree/master/keps/sig-node/1287-in-place-update-pod-resources#container-resize-policy) to specify how to go setup a resize restart policy, which indicates if a container should be restarted for the additional resource to be used. This isimportant since other than the benefit of starting the container on the same node with resize restart, it would give negligible benefit from the current Vertical Pod Scaling. 

[Issue](https://github.com/golang/go/issues/73193) opened with go so that GOMAXPROCS can be set more dynamically so that it does not need a restart. 

This inplace scaling is being added to [VPA](https://github.com/kubernetes/autoscaler/tree/master/vertical-pod-autoscaler/enhancements/7862-cpu-startup-boost). 

[Here](https://github.com/google/kube-startup-cpu-boost) is an example of this usage for increasing the CPU during the start of a JVM application. While the same cannot be done with memory. 

Nodejs should be able to handle these changes without restarts.

This was a feature which was introduced in 1.27 as alpha and has undergone a lot of changes when it came to [beta](https://github.com/kubernetes/website/blob/main/content/en/blog/_posts/2025-05-16-in-place-pod-resize-beta.md#whats-changed-between-alpha-and-beta)

Need to still figure out how this will affect binpacking the containers and how to best decide how to best utilize the nodes resources with the ability to have as little headroom as necessary but still enable this.
