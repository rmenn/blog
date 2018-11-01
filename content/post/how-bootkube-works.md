+++
title = "How bootkube works"
date = "2018-10-29"
+++

 What is bootkube

bootkube is a project in the kubernetes incubator, primarly developed by coreos, which allows you to self host the control-plane of kubernetes. Simple, isnt it?

What is self hosting

The idea of self hosting is to run the kubernetes control-plane in kubernetes leveraging kubernetes objects. That is, to run the api-server, controller-manager & the scheduler as kubernetes objects, which then enables you to manage kubernetes like any other application you run on kubernetes. What does this mean? You manage kubernetes componets using your favorite tool `kubectl`. 


Why Self host?

As mentioned above you can interact with kubernetes as it was any other application running on kubernetes, so updating kubernetes becomes as simple as `kuberctl set-image daemonset/kube-apiserver kube-apiserver=quay.io/coreos/hyperkube:v1.10.4_coreos.0`

How does bootkube work?

bootkube operates in a few modes 

- `render`
 
- `start` 

- `recover` 


`bootkube render` is used to generate sane working manifests which should idealy be used as a starting point in customising your manifests, the idea is you start with the rendered manifests and then build them to your own specification. Render is not something you should look to customize in the tool. 

`bootkube start` is used to start up the bootstrap process, this is what you wuld use to start up your cluster, we will be talking more about this

`bootkube recover` is used in recovery scenarios, when a control plane failure occurs, the tool can pull manifests from etcd and which can enable you to recover the cluster. You can read more about it here #link to recovery docs


So the first thing is that you do is to render your assets using `bootkube render <other option>`. Once rendered you would notice a couple of folders in the asset-directory, mainly `bootstrap-manifests` & `manifests`, while the other directories play an important part, these two would be the focus for now. If you look at the `manifests` directory you would see a few yamls which have been generated, which includes 

- `kube-apiserver`

- `kube-controller-manager`

- `kube-scheduler`

- `pod-checkpointer`

your CNI layer and so forth. If you look at the `Kind` in these manifests you should see daemonsets for `kube-apiserver` & `pod-checkpointer` and deployments for `kube-conteoller-manager` & `kube-scheduler`, which shows some of the control-plane components as kubernetes objects.

Now these objects can be submitted to a kubernetes cluster and your components would start running, now here is where you have a chicken-egg situation, where you need kubernetes running inorder to get your cluster up and running and this is specifically the problem that bootkube tries to solve.

Before starting off with what exactly bootkube does, you need to know what a kubelet does, a kubelet is the agent which interacts with the container runtime and the kubelet runs a pod. There are no containers run if there is no podSpec which the kubelet translates to instructions for the container runtime. All Higher level specs eventually becomes a podSpec. Below is a diagram of how a Deployment is converted into a Container.

-- ADD IMAGE --

Since the kubelet can run podSpec's bootkube relies on static pods to start up a `bootstrap` control-plane by running static pods (ADD LINK). What this means is that it runs pods on a particular server by instructing the kubelet to run them without the requirement of an apiserver. This is achieved by specifying a folder on disk with the option `--pod-manifest-path` so any podSpec placed in this path is run as a container by the kubelet. Now in the `bootstrap-manifests` directory will contain the control-plane components defined as podSpec and secrets as file mounts since we need them to be available for the static pods. 

When bootkube is started it does the following, it will move the all the files places in the `bootstrap-manifests` directory to the `pod-manifest-path` so that the kubelet will start up the control-plane as static pods. Once the files are moved to the pod-manifest-path bootkube moves into a watch mode where is checks for the bootstrap-api-server to be available. Once all the bootstrap components are available, bootkube applies all the manifests in the manifest directory as now there is an apiserver available to accept entries and write them into etcd. Once they are written to etcd the controllers take over to ensure that the manifests are running. Bootkube watches for `apiserver`, `scheduler`, `controller-manager` and the `pod-checkpointer` to be available and running ( these were hardcoded until 0.14, now you can add the pods to be watched for with a new flag `--required-pods`). Once all the components except the api-server are running ( it goes into a crash since the bootstrap-api-sever is using the port) once it reaches this state, bootkube removes all the manifests from the pod-manifest-path and exits. Since the daemonset is scheduled to run on the node, when the kubelet attempts to bring up the pod once the bootstrap-api-server is no longer running, the pod starts up and your api-server is running inside kubernetes.

Once the files are moved to the pod-manifest-path bootkube moves into a watch mode where is checks for the bootstrap-api-server to be available. Once all the bootstrap components are available, bootkube applies all the manifests in the manifest directory as now there is an api-server available to accept entries and write them into etcd. Once they are written to etcd the controllers take over to ensure that the manifests are running. Bootkube watches for api-server, scheduler, controller-manager and the pod-checkpointer to be available. Once all the components except the api-server are running ( it goes into a crash since the bootstrap-api-sever is using the port, once it reaches this state, bootkube removes all the manifests from the pod-manifest-path and exits. Since the daemonset is scheduled to run on the node, when the kubelet attempts to bring up the pod once the bootstrap-api-server is no longer running, the pod starts up and your api-server is running inside kubernetes.
There is one other component which i have mentioned but have not spoken about that is the pod-checkpointer, which would require its own post. In not so many words it creates a podSpec of the control-plane objects and stores them to disk. In the event where it detects that a component is no longer running it will move its required podSpec to the `pod-manifest-path` and ensure that the component begins to run. 


