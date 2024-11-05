+++
date = '2024-11-04T20:39:51+05:30'
draft = false
title = 'Wolfi Distroless Images'
+++

Based on this [tweet](https://x.com/rmenn/status/1847981913123819885), i decided to do this write up to show how you can create distroless images using wolfi
### What is Distroless

read more [here](https://github.com/GoogleContainerTools/distroless?tab=readme-ov-file#why-should-i-use-distroless-images)

### Why Distroless
read more [here](https://newsletter.iximiuz.com/posts/help-my-node-js-docker-image-has-python-uncovering-the-right-base-image-for-your-node-js-app)

### What is wolfi
Wolfi is a Linux undistro designed for containers, making it an excellent choice for creating distroless images. Read more [here](https://edu.chainguard.dev/open-source/wolfi/overview/)

### How to get started

You will need to install [apko](https://github.com/chainguard-dev/apko?tab=readme-ov-file#installation) and have an editor

Lets look at how distroless stacks its [images](https://labs.iximiuz.com/content/files/tutorials/how-to-choose-nodejs-container-image/__static__/google-distroless-images-hierarchy.png)

### Static Base Image
This images is for all statically compiled programs which has a directory structure, ca-certificates and tzdata

So lets setup an apko config to install these

```yaml
contents:
  keyring:
    - https://packages.wolfi.dev/os/wolfi-signing.rsa.pub
  repositories:
    - https://packages.wolfi.dev/os
  packages:
    - ca-certificates-bundle
    - tzdata
    - wolfi-baselayout
```

this installs
  * **baselayout** - this is the basic layout of the container filesystem, sets up directory structure and permissions for paths, sets root password as `*` in the passwd file so that the user cannot login, sets up `os-release` so scanners are able to identify the os and get the vulnerability db for the same 
  * **ca-certificates-bundle** - these are the cert chains for ssl connections
  * **tzdata** - is the data with which timezones are used by the OS

you can build the image with 
```bash
apko build --arch x86_64 static.yaml rmenn/static static.tar.gz
```

this will give an image which you can load a statically compiled program such as a go binary

With the apko install, there is no package manager which is added to the container, so we are closer to a distroless install.

Now we need to make sure that the container does not run with `root` user

```yaml
contents:
  keyring:
    - https://packages.wolfi.dev/os/wolfi-signing.rsa.pub
  repositories:
    - https://packages.wolfi.dev/os
  packages:
    - ca-certificates-bundle
    - tzdata
    - wolfi-baselayout
accounts:
  groups:
  - groupname: nonroot
    gid: 65532
  users:
  - username: nonroot
    uid: 65532
  run-as: nonroot
```

this will ensure that the container is run with the user `non-root`.

We need to ensure a few defaults so that the container will run as required

```yaml
contents:
  keyring:
    - https://packages.wolfi.dev/os/wolfi-signing.rsa.pub
  repositories:
    - https://packages.wolfi.dev/os
  packages:
    - ca-certificates-bundle
    - tzdata
    - wolfi-baselayout
environment:
  PATH: /usr/sbin:/sbin:/usr/bin:/bin
accounts:
  groups:
  - groupname: nonroot
    gid: 65532
  users:
  - username: nonroot
    uid: 65532
  run-as: nonroot
```

### Nodejs Runtime Image

Lets now setup `nodejs` with the above container as base

```yaml
include: static.yaml

contents:
  packages:
  - nodejs-23

accounts:
  run-as: nonroot

entrypoint:
  command: node
```

build it with 
```bash
apko build --arch x86_64 node-23.yaml rmenn/node-23 node-23.tar.gz
```

the above container is to be used to run the code and any of the nodejs containers such as `node:lts` ( glibc based ) can be used in any of the build,lint etc steps

This ensures that the container which is deployed is minimal, has only the packages required and runs with the `non-root` user which was added in the static container

### Nodejs Debug Image

Now is the question of the debug container, distroless provides this as the language container along with a shell. If running kubernetes the best way to debug would be to use the [`cdebug`](https://github.com/iximiuz/cdebug) utility but since we are talking about images lets create a nodejs debug image. 


For debugging lets add the following
  * shell
  * package manager

so that as someone who is debugging can exec into the container and install any debug tool of preference and switch the user to `root`. 

```yaml
include: node-23.yaml

contents:
  packages:
  - apk-tools
  - busybox

accounts:
  run-as: root
  
entrypoint:
  command: /bin/sh -l
 
```

build it with 
```bash
apko build --arch x86_64 node-23-debug.yaml rmenn/node-23-debug node-23-debug.tar.gz
```

this setup would allow the user to switch to the debug container which is built on top of the container which runs the code along with access to necessary tooling.


This can be used as an example and one can build out custom base images using wolfi while keeping them distroless.

{{< admonition >}}
Chainguard provides all packages installed, they have only ceased providing versioned images and are still publishing apk packages as part of the wolfi repository
{{< /admonition >}}

### Image Size
Here are the images sizes post creation

```bash
REPOSITORY                      TAG            IMAGE ID       CREATED         SIZE
rmenn/node-23                   latest-amd64   06594e5711de   13 hours ago    135MB
rmenn/node-23-debug             latest-amd64   8075f717cfc2   13 hours ago    136MB
rmenn/static                    latest-amd64   73eebc05792f   12 days ago     1.31MB
```

### Scanning the image

here is a trivy scan of the above node image

```bash
2024-11-05T12:25:04+05:30       INFO    [vuln] Vulnerability scanning is enabled
2024-11-05T12:25:04+05:30       INFO    [secret] Secret scanning is enabled
2024-11-05T12:25:04+05:30       INFO    [secret] If your scanning is slow, please try '--scanners vuln' to disable secret scanning
2024-11-05T12:25:04+05:30       INFO    [secret] Please see also https://aquasecurity.github.io/trivy/v0.56/docs/scanner/secret#recommendation for faster secret detection
2024-11-05T12:25:04+05:30       INFO    Detected OS     family="wolfi" version="20230201"
2024-11-05T12:25:04+05:30       INFO    [wolfi] Detecting vulnerabilities...    pkg_num=20
2024-11-05T12:25:04+05:30       INFO    Number of language-specific files       num=0

node-23.tar.gz (wolfi 20230201)

Total: 0 (UNKNOWN: 0, LOW: 0, MEDIUM: 0, HIGH: 0, CRITICAL: 0)
```
it has identified the OS and the packages installed. And reports 0 vulnerabilities.


### Creating a base image equivalent to alpine

Now that wolfi-base is an image which comes with a shell and packaging tools, this container image can be used to create images which are used in multi-stage build as the build container as multiple packages would be required in the build step. 


We use the static config here as well and add the `wolfi-base` package which adds busybox and apk-tools to the container, similar to what was added in the debug container but we use this meta package instead. 
```yaml
include: static.yaml
contents:
  packages:
    - wolfi-base
accounts:
  run-as: root

entrypoint:
  command: /bin/sh -l
```


Here is a [link](https://github.com/rmenn/wolfi) to all the files on github along with a plausable directory structure for multi versioned builds
