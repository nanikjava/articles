---
date: "2019-09-01"
title: "Inside minikube ISO (Work in progress)"
---


---------------------
Inside minikube .iso
---------------------

The following command will build the minikube.iso

* make out/minikube.iso 2>&1 | tee buildminikubeiso.txt
* how to build the .iso
* building .iso means creating rootfs + kernel using buildroot
* show the content of the .iso files
* extract and chroot the .iso files to show the content
