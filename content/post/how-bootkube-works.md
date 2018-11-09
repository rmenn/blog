+++
title = "How bootkube works"
date = "2018-11-02"
+++

**What is bootkube?**

bootkube is a project in the kubernetes incubator, primarly developed by coreos, which allows you to self host the control-plane of kubernetes. Simple, isnt it?


**What is self hosting?**

The idea of self hosting is to run the kubernetes control-plane in kubernetes, leveraging kubernetes objects. That is, to run the api-server, controller-manager & the scheduler as kubernetes objects, which then enables you to manage kubernetes like any other application you run on kubernetes. What does this mean? You manage kubernetes components using your favorite tool `kubectl`. 


**Why self host?**

As mentioned above you can interact with kubernetes as it was any other application running on kubernetes, so updating kubernetes becomes as simple as `kubectl set-image daemonset/kube-apiserver kube-apiserver=quay.io/coreos/hyperkube:v1.10.4_coreos.0`

**How does bootkube work?**

bootkube operates in a few modes 

- `render`
 
- `start` 

- `recover` 


`bootkube render` is used to generate sane working manifests which should idealy be used as a starting point in customising your manifests, the idea is you start with the rendered manifests and then build them to your own specification. Render is not something you should look to customize in the tool. 

`bootkube start` is used to start up the bootstrap process, this is what you would use to start up your cluster, we will be talking more about this

`bootkube recover` is used in recovery scenarios, when a control plane failure occurs, the tool can pull manifests from etcd and which can enable you to recover the cluster. You can read more about it [here][1] 


**bootkube render**

So the first thing is that you do is to render your assets using `bootkube render <other option>`. Once rendered you would notice a couple of folders in the asset-directory, mainly `bootstrap-manifests` & `manifests`, while the other directories play an important part, these two would be the focus for now.

The directories under your asset directory would look like below, the directory structure is important to bootkube

```
├── auth
├── bootstrap-manifests
├── manifests
└── tls
    └── etcd

```
If you look at the `bootstrap-manifests` directory you will see the control-plane which all have the `Kind` as `Pod`

```
.
├── bootstrap-apiserver.yaml
├── bootstrap-controller-manager.yaml
└── bootstrap-scheduler.yaml

```

If you look at the `manifests` directory you would see a few yamls which have been generated, which includes 

```
.
├── coredns-cluster-role-binding.yaml
├── coredns-cluster-role.yaml
├── coredns-config.yaml
├── coredns-deployment.yaml
├── coredns-service-account.yaml
├── coredns-service.yaml
├── csr-approver-role-binding.yaml
├── csr-bootstrap-role-binding.yaml
├── csr-renewal-role-binding.yaml
├── flannel-cfg.yaml
├── flannel-cluster-role-binding.yaml
├── flannel-cluster-role.yaml
├── flannel-sa.yaml
├── flannel.yaml
├── kube-apiserver-secret.yaml
├── kube-apiserver.yaml
├── kubeconfig-in-cluster.yaml
├── kube-controller-manager-disruption.yaml
├── kube-controller-manager-role-binding.yaml
├── kube-controller-manager-secret.yaml
├── kube-controller-manager-service-account.yaml
├── kube-controller-manager.yaml
├── kubelet-bootstrap-token.yaml
├── kube-proxy-role-binding.yaml
├── kube-proxy-sa.yaml
├── kube-proxy.yaml
├── kube-scheduler-disruption.yaml
├── kube-scheduler.yaml
├── kube-system-rbac-role-binding.yaml
├── pod-checkpointer-role-binding.yaml
├── pod-checkpointer-role.yaml
├── pod-checkpointer-sa.yaml
└── pod-checkpointer.yaml

```

your CNI layer and so forth. If you look at the `Kind` in these manifests you should see daemonsets for `kube-apiserver` & `pod-checkpointer` and deployments for `kube-conteoller-manager` & `kube-scheduler`, which shows some of the control-plane components as kubernetes objects.

Now these objects can be submitted to a kubernetes cluster and your components would start running, now here is where you have a chicken-egg situation, where you need kubernetes running inorder to get your cluster up and running and this is specifically the problem that bootkube tries to solve.


**bootkube start**

Before starting off with what exactly bootkube does, you need to know what a kubelet does. A kubelet is the agent which interacts with the container runtime and the kubelet runs a pod. There are no containers run if there is no podSpec which the kubelet translates to instructions for the container runtime. All Higher level specs eventually becomes a podSpec.


Since the kubelet can run podSpec's bootkube relies on static [pods][2] to start up a `bootstrap` control-plane by running them as such. What this means is that it runs pods on a particular server by instructing the kubelet to run them without the requirement of an apiserver. This is achieved by specifying a folder on disk with the option `--pod-manifest-path` so any podSpec placed in this path is run as a container by the kubelet. Now the `bootstrap-manifests` directory will contain the control-plane components defined as podSpec and secrets as file mounts since we need them to be available for the static pods. 


When bootkube is started it does a few things, it will move the all the files placed in the `bootstrap-manifests` directory to the `pod-manifest-path`, so that the kubelet can start up the control-plane as static pods. Once the files are moved to the pod-manifest-path bootkube moves into a watch mode where is checks for the bootstrap-apiserver to be available. Once all the bootstrap components are available, bootkube applies all the manifests in the `manifest` directory as now there is an apiserver available to accept and write them into etcd. Once they are written to etcd the controllers take over to ensure that the manifests are running. 

Once again bootkube goes into a watch mode looking the following pods `apiserver`, `scheduler`, `controller-manager` and the `pod-checkpointer` to be available and running ( these were hardcoded until 0.14, now you can add the pods to be watched for with a new flag `--required-pods`). When all the components except the apiserver are running ( as it goes into a crash since the bootstrap-apisever is using the port) once it reaches this state, bootkube removes all the manifests from the `pod-manifest-path` and exits. Since the daemonset is scheduled to run on the node, when the kubelet attempts to bring up the pod once the bootstrap-apiserver is no longer running, the pod starts up and your control-plane is running inside kubernetes.

**checkpointer**

There is one other component which i have mentioned but have not spoken about that is the pod-checkpointer, which would require its own post. In not so many words it creates a podSpec of the control-plane objects and stores them to disk. In the event where it detects that a component is no longer running it will move its required podSpec to the `pod-manifest-path` and ensure that the component begins to run. 






Thanks to nemo, gappan, hashfyre and aaron for making sure i try and get this right

Also, Happy Birthday Pa
[1]: https://github.com/kubernetes-incubator/bootkube/blob/master/Documentation/disaster-recovery.md#bootkube-recover-usage
[2]: https://kubernetes.io/docs/tasks/administer-cluster/static-pod/
