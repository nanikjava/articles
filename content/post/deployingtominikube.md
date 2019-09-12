---
date: "2019-09-01"
title: "Deploying to Minikube(Work in progress)"
author : "Nanik Tolaram (nanikjava@gmail.com)" 
---


<h1>Launching minikube</h1>


<h1>Deployment</h1>

* deploying via IDE
* deploying via CLI


---------------------
Deploying hello-world
---------------------
* 
https://kubernetes.io/docs/setup/learning-environment/minikube/

	* kubectl run hello-minikube --image=k8s.gcr.io/echoserver:1.10 --port=8080

	* kubectl expose deployment hello-minikube --type=NodePort

	* kubectl get pod

	* minikube service hello-minikube --url

