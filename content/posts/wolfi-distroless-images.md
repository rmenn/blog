+++
date = '2024-11-04T20:39:51+05:30'
draft = true
title = 'Wolfi Distroless Images'
+++

Based on this tweet, i decided to do this write up to show how you can create distroless images using wolfi


You will need to install apko and have an editor

Lets look at how distroless stacks its images

1. Static
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
  * baselayout - this is the basic layout of the container filesystem
  * ca-certificates-bundle - these are the cert chains for ssl connections
  * tzdata - is the data with which timezones are used by the OS

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
