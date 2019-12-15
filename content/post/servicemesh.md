---
date: "2019-11-30"
title: "Service Mesh Brain Dump (ALWAYS WIP)"
author : "Nanik Tolaram (nanikjava@gmail.com)" 

---
<h1> Service Mesh </h1>

* Resources
    * [Consul whitepaper](https://www.datocms-assets.com/2885/1536681707-consulwhitepaperaug2018.pdf) that explains about service mesh.\
* A service mesh is a software-driven approach to routing and segmentation. The goal is to solve the networking and security challenges of operating microservices and cloud infrastructure. Service mesh solutions bring additional benefits such as failure handling, retries, and network observability. A service mesh is an abstract concept that solves multiple service networking challenges in an integrated way.
* Following diagrams show the simplest diagram of what service mesh is all about

![ServiceMeshBasic](/media/servicemesh/servicemeshbasic.png)

![ServiceMeshBasic](/media/servicemesh/service-mesh-with-side-car.png)


* A service mesh has two key components
    * **Control Plane** --  The control plane holds the state of the system and plays a coordination role. It provides a centralized registry of where services are running, and the policies that restrict traffic. It must scale to handle tens of thousands of service instances, and efficiently update the data plane in real time. 
    * **Data Plane** -- The data plane is distributed and responsible for transmission of data between different services. It must be high-performance and integrate with the control plane.
* Following are some of the projects that perform the above 2 different planes
    * **Data planes** -- Linkerd, NGINX, HAProxy, Envoy, Traefik
    * **Control planes** -- Istio, Nelson, SmartStack
* **Ingress** -- term used for incoming request
* **Egress**  -- term used for outgoing request
* By default, proxies handle only intra-service mesh cluster traffic - between the source (upstream) and the destination (downstream) services. To expose a service which is part of service mesh to outside word, you have to enable ingress traffic. Similarly, if a service depends on an external service you may require enabling the egress traffic. 

    ![ServiceMeshBasic](/media/servicemesh/Service_Mesh_Ingress_Egress.png)

* Due to distributed nature of service mesh, a control plane or a similar centralized management utility is desirable. This elps in the management of routing tables, service discovery, and load balancing pools. 

    ![ServiceMeshBasic](/media/servicemesh/servicemeshcontrolplane.png)

* API Gateway vs Service Mesh
    * There is some overlap between API Gateway and Service mesh patterns i.e. request routing, composition, authentication, rate limiting, error handling, monitoring, etc. The primary focus of a service mesh is __service-to-service communication (internal traffic)__, whereas an API Gateway offers a single entry point for external clients such as a mobile app or a web app used by end-user to access services (external traffic).
    * Technically one can use API Gateway to facilitate internal service-to-service communication but not without introducing the latency and performance issues. As the composition of services can change over time and should be hidden from external clients - API Gateway makes sure that this complexity is hidden from the external client. 
    * It is important to note that a service mesh can use a __diverse set of protocols (RPC/gRPC/REST)__, some of which might not be friendly to external clients. 
    * API Gateway can handle multiple types of protocols and if can support protocol translation if required. There is no reason why one can not use API Gateway in front of a service mesh.
    
        ![ServiceMeshBasic](/media/servicemesh/Service-Mesh-API-Gateway.png)

<h1> Consul </h1>

* Following is a high level diagram showing how consul works as a __control plane__ capacity

![consulcontrolplane](/media/servicemesh/consulcontrolplane.png)

<h1> Istio </h1>

* Resources
    * The explanation in this article is extracted from [here](https://istio.io/blog/2019/data-plane-setup/)
    * [Indepth article about the traffic flow using envoy as sidecar](https://medium.com/faun/understanding-how-envoy-sidecar-intercept-and-route-traffic-in-istio-service-mesh-20fea2a78833)
    * An indepth [article](https://venilnoronha.io/hand-crafting-a-sidecar-proxy-and-demystifying-istio) on creating your own sidecar proxy for app running in Kubernetes.
    * [Cloud patterns](https://docs.microsoft.com/en-us/azure/architecture/patterns/)
    * Using kiali.io with Istio [presentation](https://events19.linuxfoundation.org/wp-content/uploads/2018/07/Introduction-to-Service-Mesh-with-Istio-and-Kiali-OSS-Japan-July-2019.pdf)
* Istio provides _<u>automatic injection</u>_ of sidecar proxy in Kubernetes
* In simple terms, sidecar injection is adding the configuration of additional containers to the pod template. The added containers needed for the Istio service mesh are:

    * istio-init This init container is used to setup the iptables rules so that inbound/outbound traffic will go through the sidecar proxy. An init container is different than an app container in following ways:
        * It runs before an app container is started and it always runs to completion.
        * If there are many init containers, each should complete with success before the next container is started.

  So, you can see how this type of container is perfect for a set-up or initialization job which does not need to be a part of the actual application container. In this case, istio-init does just that and sets up the iptables rules. istio-proxy This is the actual sidecar proxy (based on Envoy).
* This is done by labelling the namespace where you are deploying the app with __istio-injection=enabled__

> $ kubectl get namespaces --show-labels

{{< highlight text >}}
NAME           STATUS    AGE       LABELS
default        Active    40d       <none>
istio-dev      Active    19d       istio-injection=enabled
istio-system   Active    24d       <none>
kube-public    Active    40d       <none>
kube-system    Active    40d       <none>
{{< /highlight  >}}

* For automatic sidecar injection, Istio relies on **Mutating Admission Webhook**. Following is the instruction used to get the webhook info

> kubectl get mutatingwebhookconfiguration istio-sidecar-injector -o yaml

{{< highlight html >}}
apiVersion: admissionregistration.k8s.io/v1beta1
kind: MutatingWebhookConfiguration
metadata:
  annotations:
    kubectl.kubernetes.io/last-applied-configuration: |
      {"apiVersion":"admissionregistration.k8s.io/v1beta1","kind":"MutatingWebhookConfiguration","metadata":{"annotations":{},"labels":{"app":"istio-sidecar-injector","chart":"sidecarInjectorWebhook-1.0.1","heritage":"Tiller","release":"istio-remote"},"name":"istio-sidecar-injector","namespace":""},"webhooks":[{"clientConfig":{"caBundle":"","service":{"name":"istio-sidecar-injector","namespace":"istio-system","path":"/inject"}},"failurePolicy":"Fail","name":"sidecar-injector.istio.io","namespaceSelector":{"matchLabels":{"istio-injection":"enabled"}},"rules":[{"apiGroups":[""],"apiVersions":["v1"],"operations":["CREATE"],"resources":["pods"]}]}]}
  creationTimestamp: 2018-12-10T08:40:15Z
  generation: 2
  labels:
    app: istio-sidecar-injector
    chart: sidecarInjectorWebhook-1.0.1
    heritage: Tiller
    release: istio-remote
  name: istio-sidecar-injector
  .....
webhooks:
- clientConfig:
    service:
      name: istio-sidecar-injector
      namespace: istio-system
      path: /inject
  name: sidecar-injector.istio.io
  namespaceSelector:
    matchLabels:
      istio-injection: enabled
  rules:
  - apiGroups:
    - ""
    apiVersions:
    - v1
    operations:
    - CREATE
    resources:
    - pods
{{< /highlight  >}}

* Can see the webhook **namespaceSelector** label that is matched for sidecar injection with the label **istio-injection: enabled**.
* The 'app' that is intercepting this request is as follows

>  kubectl get svc --namespace=istio-system | grep sidecar-injector

> istio-sidecar-injector   ClusterIP   10.102.70.184   <none>        443/TCP             24d

* Now that we are clear about how a sidecar container and an init container are injected into an application manifest, how does the sidecar proxy grab the inbound and outbound traffic to and from the container? We did briefly mention that it is done by setting up the **iptable** rules within the pod namespace, which in turn is done by the istio-init container. Now, it is time to verify what actually gets updated within the namespace.

Let’s get into the application pod namespace we deployed in the previous section and looked at the configured iptables. I am going to show an example using __nsenter__. Alternatively, you can enter the container in a privileged mode to see the same information. For folks without access to the nodes, using exec to get into the sidecar and running iptables is more practical.

Get the pid of the container that has the sidecar proxy running (normally this is our app container)

> docker inspect b8de099d3510 --format '{{ .State.Pid }}'

enter into the container and execute the command

> nsenter -t 4215 -n iptables -t nat -S

The output will be as follows

{{< highlight html >}}
-P PREROUTING ACCEPT
-P INPUT ACCEPT
-P OUTPUT ACCEPT
-P POSTROUTING ACCEPT
-N ISTIO_INBOUND
-N ISTIO_IN_REDIRECT
-N ISTIO_OUTPUT
-N ISTIO_REDIRECT
-A PREROUTING -p tcp -j ISTIO_INBOUND
-A OUTPUT -p tcp -j ISTIO_OUTPUT
-A ISTIO_INBOUND -p tcp -m tcp --dport 80 -j ISTIO_IN_REDIRECT
-A ISTIO_IN_REDIRECT -p tcp -j REDIRECT --to-ports 15001
-A ISTIO_OUTPUT ! -d 127.0.0.1/32 -o lo -j ISTIO_REDIRECT
-A ISTIO_OUTPUT -m owner --uid-owner 1337 -j RETURN
-A ISTIO_OUTPUT -m owner --gid-owner 1337 -j RETURN
-A ISTIO_OUTPUT -d 127.0.0.1/32 -j RETURN
-A ISTIO_OUTPUT -j ISTIO_REDIRECT
-A ISTIO_REDIRECT -p tcp -j REDIRECT --to-ports 15001
{{< /highlight  >}}

* Internal diagrams

![consulcontrolplane](/media/servicemesh/istioexpanded.png)


* Following are resources that gives detail explanation on working with Istio
    * https://openliberty.io/guides/istio-intro.html#deploying-version-2-of-the-system-microservice




