---
date: "2019-09-05"
title: "How minikube works"
---


------------------
How minikube works
------------------
* when 'start' parameter is trigger it will execute the vboxmanage to create VM
* different vboxmanage commands are executed to the vm to configure it 
* different services are configured and run inside the VM
* different things that can be done to the vm
	- ssh into it
	- 
---------------------
Inside minikube .iso
---------------------




The following command will build the minikube.iso

	make out/minikube.iso 2>&1 | tee buildminikubeiso.txt
* how to build the .iso
* building .iso means creating rootfs + kernel using buildroot
* show the content of the .iso files
* extract and chroot the .iso files to show the content

---------------------
Deploying hello-world
---------------------
* 
https://kubernetes.io/docs/setup/learning-environment/minikube/

	* kubectl run hello-minikube --image=k8s.gcr.io/echoserver:1.10 --port=8080

	* kubectl expose deployment hello-minikube --type=NodePort

	* kubectl get pod

	* minikube service hello-minikube --url

