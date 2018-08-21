+++
date = "2017-04-26T19:53:09+05:30"
title = "Running bootkube on OSX"

+++

I'd heard of bootkube and wanted to try it out, I also really wanted to run this locally on my MacBook. I didn't want to run it on Virtual Box, this was when I remembered [kubesolo][1] which used the xhvye virtualization to stand up a kubernetes cluster on your MacBook. I then looked and found [corectl][2], which allows you to run a coreos vm on xhyve. It took me a day or so to figure out how exactly I needed to bring up the cluster. 

I'd like to share the process on how I got this done. I took the lastest bootkube version at the time of writing (0.4.1 - released less than 24 hours ago). Coreos is what we run kubernetes on, so having a playground that I could try things out was a big bonus. You can check out the repo where I have put in a few things to get things running. I've hardcoded a lot of things, you can change all the config on the script. I have tried to put out as much detail as I can on the readme, check it out [here][3].

There maybe a lot of errors on the script, its probably not the most efficient script, so be gentle.

[1]:https://github.com/TheNewNormal/kube-solo-osx
[2]:https://github.com/TheNewNormal/corectl
[3]:https://github.com/rmenn/bootkube-osx
