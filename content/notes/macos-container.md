+++
date = '2025-06-10T07:40:03+05:30'
draft = false
title = 'Macos Container'
+++


Apple just announced a way to run containers on macos. I dont believe its native, since it uses a kata vm to run the linux vm and then provides an api server to run these containers. 

I have run them on macos 15, while all the [features](https://github.com/apple/container/blob/main/docs/technical-overview.md#macos-15-limitations) are said to be available on macos 26. It runs aarch64 images

```
❯ container ls
Error: internalError: "failed to list containers" (cause: "interrupted: "XPC connection error: Connection invalid"")
❯ container system start
Verifying apiserver is running...
Installing base container filesystem...
No default kernel configured.
Install the recommended default kernel from [https://github.com/kata-containers/kata-containers/releases/download/3.17.0/kata-static-3.17.0-arm64.tar.xz]? [Y/n]: y
Installing kernel...
❯ container ls
ID  IMAGE  OS  ARCH  STATE  ADDR
❯ container run -it alpine
/ # uname -a
Linux e23cf733-affc-4557-ba99-24cbf4ecf39f 6.12.28 #1 SMP Tue May 20 15:19:05 UTC 2025 aarch64 Linux
/ # ^C
❯ container images ls
NAME    TAG     DIGEST
alpine  latest  8a1f59ffb675680d47db6337...
```

Host networking also needs a work around at this [time](https://github.com/apple/container/blob/main/docs/technical-overview.md#container-to-host-networking)

sounds like a starting point, but still far away from being a daily driver in comparission to docker, podman, orbstack or even colima. 
