---
title: "The Controller Conundrum"
date: 2020-11-20T21:22:09+05:30
draft: false
---

tl;dr - Gave up writing a controller and got similar functionality with 6 lines of bash

Recently i began planning the split of auto scalling groups for a change we were planning to introduce, this involved the nodes having certain taints & labels so that the right workload gets placed on them. 

So this involved two things one is a couple of AWS ASGs and spotinst, so i needed something which worked on all nodes

Thus began my brain thinking up on how i can solve this via a controller. My brain has been rewired to think of any problem with a contoller/operator solution. 

I fired up my editor and terminal. Started off with `kube-builder`. 

Started wrtiting down things in comments.

```go
// New node
// Taint with NoSchedule & NoExecture
// Query AWS find tenency 
// If not Spot Query ASG membership
// ...
// ...
// Add Label and Taint
```

As i started going thru the aws api calls, so i could know what aws iam role i needed to create for the controller it struct me that i am taking the long way around.

I just ended up adding a few lines of bash inorder to get a very basic functionality that i required to get thru this. I fixed a couple of tags which can be added to terraform and spotinst. `aws.eks.node/labels` & `aws.eks.node/taints`, used the basic describe tags api to get the associated tags and then pass them onto the the eks `bootstrap.sh` script. 

Without trying to explain things further here is a snippet you can use

```bash
INSTANCE_ID=`wget -qO- http://instance-data/latest/meta-data/instance-id`
REGION=`wget -qO- http://instance-data/latest/meta-data/placement/availability-zone | sed 's/.$//'`
EKS_LABELS=`aws ec2 describe-tags --region $REGION --filter "Name=resource-id,Values=$INSTANCE_ID" --output=json | jq --raw-output '.Tags[] | select(.Key=="ft/label") | .Value'`
#EKS_LABELS=""
EKS_TAINTS=`aws ec2 describe-tags --region $REGION --filter "Name=resource-id,Values=$INSTANCE_ID" --output=json | jq --raw-output '.Tags[] | select(.Key=="ft/taint") | .Value'`
EXTRA_ARGS=""
if [ ! -z "$EKS_TAINTS" ]; then EXTRA_ARGS=$EXTRA_ARGS" --register-with-taints=$EKS_TAINTS"; fi
if [ ! -z "$EKS_LABELS" ]; then EXTRA_ARGS=$EXTRA_ARGS" --node-labels=$EKS_LABELS"; fi
/etc/eks/bootstrap.sh ...(truncated)..... --kubelet-extra-args $EXRTA_ARGS $CLUSTER_NAME
```

Im sure this seems basic but i was trying to put down my mindset rather than post the solution.

There just might be an easy way to do things, might as well use that. 


Before i conclude, would the controller have been useful - quite possibly. 
Given the time constraint do i like the bash solution - YES!

Cheers
