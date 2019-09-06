---
date: "2019-09-01"
title: "Deploying to Minikube(Work in progress)"
---
---------------------
Deploying hello-world
---------------------
* 
https://kubernetes.io/docs/setup/learning-environment/minikube/

	* kubectl run hello-minikube --image=k8s.gcr.io/echoserver:1.10 --port=8080

	* kubectl expose deployment hello-minikube --type=NodePort

	* kubectl get pod

	* minikube service hello-minikube --url

