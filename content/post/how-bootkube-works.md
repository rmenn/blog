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

How does bootkube work?

bootkube operates in a few modes `render`, `start` & `recover`. 


`bootkube render` is used to generate sane working manifests which should idealy be used as a starting point in customising your manifests, the idea is you start with the rendered manifests and then build them to your own specification. Render is not something you should look to customize in the tool. 

`bootkube start` is used to start up the bootstrap process, this is what you wuld use to start up your cluster, we will be talking more about this

`bootkube recover` is used in recovery scenarios, when a control plane failure occurs, the tool can pull manifests from etcd and which can enable you to recover the cluster. You can read more about it here #link to recovery docs


So the first thing that you do is to render your assets using `bootkube render <other option>`. Once rendered you would notice a couple of folders in the asset-directory, mainly `bootstrap-manifests` & `manifests`, while the other directories play an important part, these two would be the focus for now. If you look at the `manifests` directory you would see a few yamls which have been generated, which includes `kube-apiserver`, `kube-controller-manager`, `kube-scheduler`, `pod-checkpointer`, your CNI layer and so forth. If you look at the `Kind` in these manifests you should see daemonsets for `kube-apiserver` & `pod-checkpointer` and deployments for `kube-conteoller-manager` & `kube-scheduler`, which shows some of the control-plane components as kubernetes objects.

Now these objects can be submitted to a kubernetes cluster and your components would start running, now here is where you have a chicken-egg situation, where you need kubernetes running inorder to get your cluster up and running and this is specifically the problem that bootkube tries to solve. 
