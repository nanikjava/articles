---
date: "2019-09-01"
title: "Deploying to Minikube(Work in progress)"
author : "Nanik Tolaram (nanikjava@gmail.com)" 
---


We will deploy 2 different projects in this article. One is a simple single command line application and another is a web app that requires database. We will be using the skaffold project as it will make it easy to do continous deployment

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



K8 Tools
--------
* skaffold - continues local development/deployment in k8 for golang
* k8guard  - example to show deploying different applications inside k8 ( https://k8guard.github.io/deploy/minikube/ )
* ko       - Build and deploy Go applications on Kubernetes ( https://github.com/google/ko )




