---
date: "2019-09-01"
title: "Kubernetes Brain Dump (ALWAYS WIP)"
author : "Nanik Tolaram (nanikjava@gmail.com)" 

---
* Linux Network Namespaces is one of the important feature in Linux Kernel that are leveraged by containers like docker and kubernetes. It is used in the CNI (Container Network Interface).
* References
	- https://blogs.igalia.com/dpino/2016/04/10/network-namespaces/
	- https://github.com/containernetworking/cni/blob/master/SPEC.md
	- https://github.com/containernetworking/plugins
	- https://medium.com/@vikram.fugro/container-networking-interface-aka-cni-bdfe23f865cf --> show how the CNI works using the containetworking github.com account
	- https://www.cncf.io/blog/2017/05/23/cncf-hosts-container-networking-interface-cni/ --> specification and implementation of CNI
	- https://www.weave.works/docs/net/latest/kubernetes/kube-addon/ --> weavenet is network plugin for docker for variety of things (service discovery, etc), this website outlined how to use it as inside kubernetes 

- Key points
	* A node is a VM or a physical computer that serves as a worker machine in a Kubernetes cluster. 
	* Each node has a Kubelet, which is an agent for managing the node and communicating with the Kubernetes master. The node should also have tools for handling container operations, such as Docker or rkt.
	* A Kubernetes cluster that handles production traffic should have a minimum of three nodes.
	* The nodes communicate with the master using the Kubernetes API. End users can also use the Kubernetes API directly to interact with the cluster.
	* A Kubernetes cluster can be deployed on either physical or virtual machines.
	* Once you've created a Deployment, the Kubernetes master schedules mentioned application instances onto individual Nodes in the cluster.
	* Once the application instances are created, a Kubernetes Deployment Controller continuously monitors those instances. If the Node hosting an instance goes down or is deleted, the Deployment controller replaces the instance with an instance on another Node in the cluster. This provides a self-healing mechanism to address machine failure or maintenance.
	* A Pod is a Kubernetes abstraction that represents a group of one or more application containers (such as Docker or rkt), and some shared resources for those containers. Those resources include:
		- Shared storage, as Volumes
		- Networking, as a unique cluster IP address
		- Information about how to run each container, such as the container image version or specific ports to use
		- The containers in a Pod share an IP Address and port space, are always co-located and co-scheduled, and run in a shared context on the same Node.
	* A Pod always runs on a Node
		- A Node is a worker machine in Kubernetes and may be either a virtual or a physical machine, depending on the cluster. 
		- Each Node is managed by the Master. 
		- A Node can have multiple pods, and the Kubernetes master automatically handles scheduling the pods across the Nodes in the cluster. 
		- The Master's automatic scheduling takes into account the available resources on each Node.
		- Every Kubernetes Node runs at least:
			** Kubelet, a process responsible for communication between the Kubernetes Master and the Node; it manages the Pods and the containers running on a machine.
			** A container runtime (like Docker, rkt) responsible for pulling the container image from a registry, unpacking the container, and running the application.
			** Containers should only be scheduled together in a single Pod if they are tightly coupled and need to share resources such as disk.
		
			![ContainersPods](https://d33wubrfki0l68.cloudfront.net/5cb72d407cbe2755e581b6de757e0d81760d5b86/a9df9/docs/tutorials/kubernetes-basics/public/images/module_03_nodes.svg)

	* The Kubernetes Master is a collection of three processes that run on a single node in your cluster, which is designated as the master node. Those processes are: kube-apiserver, kube-controller-manager and kube-scheduler.
		- Each individual non-master node in your cluster runs two processes:
			- kubelet, which communicates with the Kubernetes Master.
			- kube-proxy, a network proxy which reflects Kubernetes networking services on each node.

	* Services
		- A Service in Kubernetes is an abstraction which defines a logical set of Pods and a policy by which to access them. Services enable a loose coupling between dependent Pods. A Service is defined using YAML (preferred) or JSON, like all Kubernetes objects. The set of Pods targeted by a Service is usually determined by a LabelSelector (see below for why you might want a Service without including selector in the spec).
		- Services can be exposed in different ways by specifying a type in the ServiceSpec:
  			- ClusterIP (default) - Exposes the Service on an internal IP in the cluster. This type makes the Service only reachable from within the cluster.
			- NodePort - Exposes the Service on the same port of each selected Node in the cluster using NAT. Makes a Service accessible from outside the cluster using <NodeIP>:<NodePort>. Superset of ClusterIP.
			- LoadBalancer - Creates an external load balancer in the current cloud (if supported) and assigns a fixed, external IP to the Service. Superset of NodePort.
			- ExternalName - Exposes the Service using an arbitrary name (specified by externalName in the spec) by returning a CNAME record with the name. No proxy is used. This type requires v1.7 or higher of kube-dns.

		- Services match a set of Pods using labels and selectors, a grouping primitive that allows logical operation on objects in Kubernetes. Labels are key/value pairs attached to objects and can be used in any number of ways:

			** Designate objects for development, test, and production
			** Embed version tags
			** Classify an object using tags

			![Services](https://d33wubrfki0l68.cloudfront.net/cc38b0f3c0fd94e66495e3a4198f2096cdecd3d5/ace10/docs/tutorials/kubernetes-basics/public/images/module_04_services.svg)
			![Services](https://d33wubrfki0l68.cloudfront.net/b964c59cdc1979dd4e1904c25f43745564ef6bee/f3351/docs/tutorials/kubernetes-basics/public/images/module_04_labels.svg)



	* Once you’ve set your desired state, the Kubernetes Control Plane makes the cluster’s current state match the desired state via the Pod Lifecycle Event Generator (PLEG). The Kubernetes Control Plane consists of a collection of processes running on your cluster:
		- The various parts of the Kubernetes Control Plane, such as the Kubernetes Master and kubelet processes, govern how Kubernetes communicates with your cluster. The Control Plane maintains a record of all of the Kubernetes Objects in the system, and runs continuous control loops to manage those objects’ state. At any given time, the Control Plane’s control loops will respond to changes in the cluster and work to make the actual state of all the objects in the system match the desired state that you provided.
	
	* Master Components
		-Master components provide the cluster’s control plane. Master components make global decisions about the cluster (for example, scheduling), and they detect and respond to cluster events (for example, starting up a new pod when a deployment’s replicas field is unsatisfied).  For simplicity, set up scripts typically start all master components on the same machine, and do not run user containers on this machine. See Building High-Availability Clusters for an example multi-master-VM setup.

			* kube-apiserver - Component on the master that exposes the Kubernetes API. It is the front-end for the Kubernetes control plane.

			* ecd - Consistent and highly-available key value store used as Kubernetes’ backing store for all cluster data.

			* kube-scheduler - Component on the master that watches newly created pods that have no node assigned, and selects a node for them to run on.

			* kube-controller-manager - Component on the master that runs controllers.
		
			* Logically, each controller is a separate process, but to reduce complexity, they are all compiled into a single binary and run in a single process. These controllers include:
				- Node Controller: Responsible for noticing and responding when nodes go down.
				- Replication Controller: Responsible for maintaining the correct number of pods for every replication controller object in the system.
				- Endpoints Controller: Populates the Endpoints object (that is, joins Services & Pods).
				- Service Account & Token Controllers: Create default accounts and API access tokens for new namespaces.


			* cloud-controller-manager - cloud-controller-manager runs controllers that interact with the underlying cloud providers. The cloud-controller-manager binary is an alpha feature introduced in Kubernetes release 1.6.

				- cloud-controller-manager runs cloud-provider-specific controller loops only. You must disable these controller loops in the kube-controller-manager. You can disable the controller loops by setting the --cloud-provider flag to external when starting the kube-controller-manager. cloud-controller-manager allows the cloud vendor’s code and the Kubernetes code to evolve independently of each other. In prior releases, the core Kubernetes code was dependent upon cloud-provider-specific code for functionality. In future releases, code specific to cloud vendors should be maintained by the cloud vendor themselves, and linked to cloud-controller-manager while running Kubernetes.

				- The following controllers have cloud provider dependencies:

					[x] Node Controller: For checking the cloud provider to determine if a node has been deleted in the cloud after it stops responding
					[x] Route Controller: For setting up routes in the underlying cloud infrastructure
					[x] Service Controller: For creating, updating and deleting cloud provider load balancers
					[x] Volume Controller: For creating, attaching, and mounting volumes, and interacting with the cloud provider to orchestrate volumes


	* Node Components
		- Node components run on every node, maintaining running pods and providing the Kubernetes runtime environment.

			* kubelet - An agent that runs on each node in the cluster. It makes sure that containers are running in a pod.  The kubelet takes a set of PodSpecs that are provided through various mechanisms and ensures that the containers described in those PodSpecs are running and healthy. The kubelet doesn’t manage containers which were not created by Kubernetes.

			* kube-proxy - kube-proxy is a network proxy that runs on each node in your cluster, implementing part of the Kubernetes Service concept. kube-proxy maintains network rules on nodes. These network rules allow network communication to your Pods from network sessions inside or outside of your cluster. kube-proxy uses the operating system packet filtering layer if there is one and it’s available. Otherwise, kube-proxy forwards the traffic itself.

			* Container Runtime - The container runtime is the software that is responsible for running containers.

	* Addons
		- Addons use Kubernetes resources (DaemonSet, Deployment, etc) to implement cluster features. Because these are providing cluster-level features, namespaced resources for addons belong within the kube-system namespace.

			* DNS - While the other addons are not strictly required, all Kubernetes clusters should have cluster DNS, as many examples rely on it. Cluster DNS is a DNS server, in addition to the other DNS server(s) in your environment, which serves DNS records for Kubernetes services. Containers started by Kubernetes automatically include this DNS server in their DNS searches.

			* Web UI (Dashboard) - Dashboard is a general purpose, web-based UI for Kubernetes clusters. It allows users to manage and troubleshoot applications running in the cluster, as well as the cluster itself.

			* Container Resource Monitoring - Container Resource Monitoring records generic time-series metrics about containers in a central database, and provides a UI for browsing that data.

			* Cluster-level Logging - Cluster-level logging mechanism is responsible for saving container logs to a central log store with search/browsing interface.



	* The basic Kubernetes objects include:

		- Pod
		- Service
		- Volume
		- Namespace


* etcd
	- To communicate to etcd that are running inside minikube we need to do the following
		* minikube ssh 
		* /hosthome/nanik/Downloads/temp/packages/src/go.etcd.io/etcd/etcdctl/etcdctl --cacert=/var/lib/minikube/certs/etcd/ca.crt --key=/var/lib/minikube/certs/etcd/ca.key --cert=/var/lib/minikube/certs/etcd/ca.crt get  --prefix=true ""
		* /hostname --> default mount volume created by minikube to access host directory.
		* ETCDCTL_API=3  /hosthome/nanik/Downloads/temp/packages/src/go.etcd.io/etcd/etcdctl/etcdctl --cacert=/var/lib/minikube/certs/etcd/ca.crt --key=/var/lib/minikube/certs/etcd/ca.key --cert=/var/lib/minikube/certs/etcd/ca.crt get  --from-key '' --keys-only --> will get all the keys stored inside


					/registry/apiregistration.k8s.io/apiservices/v1.

					/registry/apiregistration.k8s.io/apiservices/v1.admissionregistration.k8s.io

					/registry/apiregistration.k8s.io/apiservices/v1.apiextensions.k8s.io

					/registry/apiregistration.k8s.io/apiservices/v1.apps

					/registry/apiregistration.k8s.io/apiservices/v1.authentication.k8s.io

					/registry/apiregistration.k8s.io/apiservices/v1.authorization.k8s.io

					/registry/apiregistration.k8s.io/apiservices/v1.autoscaling

					/registry/apiregistration.k8s.io/apiservices/v1.batch

					/registry/apiregistration.k8s.io/apiservices/v1.coordination.k8s.io

					/registry/apiregistration.k8s.io/apiservices/v1.networking.k8s.io

					/registry/apiregistration.k8s.io/apiservices/v1.rbac.authorization.k8s.io

					/registry/apiregistration.k8s.io/apiservices/v1.scheduling.k8s.io

					/registry/apiregistration.k8s.io/apiservices/v1.storage.k8s.io

					/registry/apiregistration.k8s.io/apiservices/v1beta1.admissionregistration.k8s.io

					/registry/apiregistration.k8s.io/apiservices/v1beta1.apiextensions.k8s.io

					/registry/apiregistration.k8s.io/apiservices/v1beta1.authentication.k8s.io

					/registry/apiregistration.k8s.io/apiservices/v1beta1.authorization.k8s.io

					/registry/apiregistration.k8s.io/apiservices/v1beta1.batch

					/registry/apiregistration.k8s.io/apiservices/v1beta1.certificates.k8s.io

					/registry/apiregistration.k8s.io/apiservices/v1beta1.coordination.k8s.io

					/registry/apiregistration.k8s.io/apiservices/v1beta1.events.k8s.io

					/registry/apiregistration.k8s.io/apiservices/v1beta1.extensions

					/registry/apiregistration.k8s.io/apiservices/v1beta1.networking.k8s.io

					/registry/apiregistration.k8s.io/apiservices/v1beta1.node.k8s.io

					/registry/apiregistration.k8s.io/apiservices/v1beta1.policy

					/registry/apiregistration.k8s.io/apiservices/v1beta1.rbac.authorization.k8s.io

					/registry/apiregistration.k8s.io/apiservices/v1beta1.scheduling.k8s.io

					/registry/apiregistration.k8s.io/apiservices/v1beta1.storage.k8s.io

					/registry/apiregistration.k8s.io/apiservices/v2beta1.autoscaling

					/registry/apiregistration.k8s.io/apiservices/v2beta2.autoscaling

					/registry/clusterrolebindings/cluster-admin

					/registry/clusterrolebindings/kubeadm:kubelet-bootstrap

					/registry/clusterrolebindings/kubeadm:node-autoapprove-bootstrap

					/registry/clusterrolebindings/kubeadm:node-autoapprove-certificate-rotation

					/registry/clusterrolebindings/kubeadm:node-proxier

					/registry/clusterrolebindings/minikube-rbac

					/registry/clusterrolebindings/storage-provisioner

					/registry/clusterrolebindings/system:basic-user

					/registry/clusterrolebindings/system:controller:attachdetach-controller

					/registry/clusterrolebindings/system:controller:certificate-controller

					/registry/clusterrolebindings/system:controller:clusterrole-aggregation-controller

					/registry/clusterrolebindings/system:controller:cronjob-controller

					/registry/clusterrolebindings/system:controller:daemon-set-controller

					/registry/clusterrolebindings/system:controller:deployment-controller

					/registry/clusterrolebindings/system:controller:disruption-controller

					/registry/clusterrolebindings/system:controller:endpoint-controller

					/registry/clusterrolebindings/system:controller:expand-controller

					/registry/clusterrolebindings/system:controller:generic-garbage-collector

					/registry/clusterrolebindings/system:controller:horizontal-pod-autoscaler

					/registry/clusterrolebindings/system:controller:job-controller

					/registry/clusterrolebindings/system:controller:namespace-controller

					/registry/clusterrolebindings/system:controller:node-controller

					/registry/clusterrolebindings/system:controller:persistent-volume-binder

					/registry/clusterrolebindings/system:controller:pod-garbage-collector

					/registry/clusterrolebindings/system:controller:pv-protection-controller

					/registry/clusterrolebindings/system:controller:pvc-protection-controller

					/registry/clusterrolebindings/system:controller:replicaset-controller

					/registry/clusterrolebindings/system:controller:replication-controller

					/registry/clusterrolebindings/system:controller:resourcequota-controller

					/registry/clusterrolebindings/system:controller:route-controller

					/registry/clusterrolebindings/system:controller:service-account-controller

					/registry/clusterrolebindings/system:controller:service-controller

					/registry/clusterrolebindings/system:controller:statefulset-controller

					/registry/clusterrolebindings/system:controller:ttl-controller

					/registry/clusterrolebindings/system:coredns

					/registry/clusterrolebindings/system:discovery

					/registry/clusterrolebindings/system:kube-controller-manager

					/registry/clusterrolebindings/system:kube-dns

					/registry/clusterrolebindings/system:kube-scheduler

					/registry/clusterrolebindings/system:node

					/registry/clusterrolebindings/system:node-proxier

					/registry/clusterrolebindings/system:public-info-viewer

					/registry/clusterrolebindings/system:volume-scheduler

					/registry/clusterroles/admin

					/registry/clusterroles/cluster-admin

					/registry/clusterroles/edit

					/registry/clusterroles/system:aggregate-to-admin

					/registry/clusterroles/system:aggregate-to-edit

					/registry/clusterroles/system:aggregate-to-view

					/registry/clusterroles/system:auth-delegator

					/registry/clusterroles/system:basic-user

					/registry/clusterroles/system:certificates.k8s.io:certificatesigningrequests:nodeclient

					/registry/clusterroles/system:certificates.k8s.io:certificatesigningrequests:selfnodeclient

					/registry/clusterroles/system:controller:attachdetach-controller

					/registry/clusterroles/system:controller:certificate-controller

					/registry/clusterroles/system:controller:clusterrole-aggregation-controller

					/registry/clusterroles/system:controller:cronjob-controller

					/registry/clusterroles/system:controller:daemon-set-controller

					/registry/clusterroles/system:controller:deployment-controller

					/registry/clusterroles/system:controller:disruption-controller

					/registry/clusterroles/system:controller:endpoint-controller

					/registry/clusterroles/system:controller:expand-controller

					/registry/clusterroles/system:controller:generic-garbage-collector

					/registry/clusterroles/system:controller:horizontal-pod-autoscaler

					/registry/clusterroles/system:controller:job-controller

					/registry/clusterroles/system:controller:namespace-controller

					/registry/clusterroles/system:controller:node-controller

					/registry/clusterroles/system:controller:persistent-volume-binder

					/registry/clusterroles/system:controller:pod-garbage-collector

					/registry/clusterroles/system:controller:pv-protection-controller

					/registry/clusterroles/system:controller:pvc-protection-controller

					/registry/clusterroles/system:controller:replicaset-controller

					/registry/clusterroles/system:controller:replication-controller

					/registry/clusterroles/system:controller:resourcequota-controller

					/registry/clusterroles/system:controller:route-controller

					/registry/clusterroles/system:controller:service-account-controller

					/registry/clusterroles/system:controller:service-controller

					/registry/clusterroles/system:controller:statefulset-controller

					/registry/clusterroles/system:controller:ttl-controller

					/registry/clusterroles/system:coredns

					/registry/clusterroles/system:csi-external-attacher

					/registry/clusterroles/system:csi-external-provisioner

					/registry/clusterroles/system:discovery

					/registry/clusterroles/system:heapster

					/registry/clusterroles/system:kube-aggregator

					/registry/clusterroles/system:kube-controller-manager

					/registry/clusterroles/system:kube-dns

					/registry/clusterroles/system:kube-scheduler

					/registry/clusterroles/system:kubelet-api-admin

					/registry/clusterroles/system:node

					/registry/clusterroles/system:node-bootstrapper

					/registry/clusterroles/system:node-problem-detector

					/registry/clusterroles/system:node-proxier

					/registry/clusterroles/system:persistent-volume-provisioner

					/registry/clusterroles/system:public-info-viewer

					/registry/clusterroles/system:volume-scheduler

					/registry/clusterroles/view

					/registry/configmaps/kube-public/cluster-info

					/registry/configmaps/kube-system/coredns

					/registry/configmaps/kube-system/extension-apiserver-authentication

					/registry/configmaps/kube-system/kube-proxy

					/registry/configmaps/kube-system/kubeadm-config

					/registry/configmaps/kube-system/kubelet-config-1.16

					/registry/controllerrevisions/kube-system/kube-proxy-68594d95c

					/registry/daemonsets/kube-system/kube-proxy

					/registry/deployments/default/hello-node

					/registry/deployments/kube-system/coredns

					/registry/events/default/hello-node-7676b5fb8d-m5286.15c75558016b01db

					/registry/events/default/hello-node-7676b5fb8d-m5286.15c7555860a8a944

					/registry/events/default/hello-node-7676b5fb8d-m5286.15c755660d32db77

					/registry/events/default/hello-node-7676b5fb8d-m5286.15c7556615f7efc7

					/registry/events/default/hello-node-7676b5fb8d-m5286.15c755661e3260a8

					/registry/events/default/hello-node-7676b5fb8d.15c7555800602772

					/registry/events/default/hello-node.15c75557ff3eff90

					/registry/leases/kube-node-lease/minikube

					/registry/masterleases/192.168.99.119

					/registry/minions/minikube

					/registry/namespaces/default

					/registry/namespaces/kube-node-lease

					/registry/namespaces/kube-public

					/registry/namespaces/kube-system

					/registry/pods/default/hello-node-7676b5fb8d-m5286

					/registry/pods/kube-system/coredns-5644d7b6d9-2269l

					/registry/pods/kube-system/coredns-5644d7b6d9-jlxpn

					/registry/pods/kube-system/etcd-minikube

					/registry/pods/kube-system/kube-addon-manager-minikube

					/registry/pods/kube-system/kube-apiserver-minikube

					/registry/pods/kube-system/kube-controller-manager-minikube

					/registry/pods/kube-system/kube-proxy-phnsk

					/registry/pods/kube-system/kube-scheduler-minikube

					/registry/pods/kube-system/storage-provisioner

					/registry/priorityclasses/system-cluster-critical

					/registry/priorityclasses/system-node-critical

					/registry/ranges/serviceips

					/registry/ranges/servicenodeports

					/registry/replicasets/default/hello-node-7676b5fb8d

					/registry/replicasets/kube-system/coredns-5644d7b6d9

					/registry/rolebindings/kube-public/kubeadm:bootstrap-signer-clusterinfo

					/registry/rolebindings/kube-public/system:controller:bootstrap-signer

					/registry/rolebindings/kube-system/kube-proxy

					/registry/rolebindings/kube-system/kubeadm:kubelet-config-1.16

					/registry/rolebindings/kube-system/kubeadm:nodes-kubeadm-config

					/registry/rolebindings/kube-system/system::extension-apiserver-authentication-reader

					/registry/rolebindings/kube-system/system::leader-locking-kube-controller-manager

					/registry/rolebindings/kube-system/system::leader-locking-kube-scheduler

					/registry/rolebindings/kube-system/system:controller:bootstrap-signer

					/registry/rolebindings/kube-system/system:controller:cloud-provider

					/registry/rolebindings/kube-system/system:controller:token-cleaner

					/registry/roles/kube-public/kubeadm:bootstrap-signer-clusterinfo

					/registry/roles/kube-public/system:controller:bootstrap-signer

					/registry/roles/kube-system/extension-apiserver-authentication-reader

					/registry/roles/kube-system/kube-proxy

					/registry/roles/kube-system/kubeadm:kubelet-config-1.16

					/registry/roles/kube-system/kubeadm:nodes-kubeadm-config

					/registry/roles/kube-system/system::leader-locking-kube-controller-manager

					/registry/roles/kube-system/system::leader-locking-kube-scheduler

					/registry/roles/kube-system/system:controller:bootstrap-signer

					/registry/roles/kube-system/system:controller:cloud-provider

					/registry/roles/kube-system/system:controller:token-cleaner

					/registry/secrets/default/default-token-p2tsg

					/registry/secrets/kube-node-lease/default-token-d9v9t

					/registry/secrets/kube-public/default-token-jm6qq

					/registry/secrets/kube-system/attachdetach-controller-token-zplrk

					/registry/secrets/kube-system/bootstrap-signer-token-7mljm

					/registry/secrets/kube-system/bootstrap-token-u7t69o

					/registry/secrets/kube-system/certificate-controller-token-cpqnf

					/registry/secrets/kube-system/clusterrole-aggregation-controller-token-mz4xc

					/registry/secrets/kube-system/coredns-token-qfsk7

					/registry/secrets/kube-system/cronjob-controller-token-6nlcw

					/registry/secrets/kube-system/daemon-set-controller-token-cglwc

					/registry/secrets/kube-system/default-token-ntnd8

					/registry/secrets/kube-system/deployment-controller-token-lzlgw

					/registry/secrets/kube-system/disruption-controller-token-rgmxg

					/registry/secrets/kube-system/endpoint-controller-token-c4khp

					/registry/secrets/kube-system/expand-controller-token-mg9wd

					/registry/secrets/kube-system/generic-garbage-collector-token-hjc5m

					/registry/secrets/kube-system/horizontal-pod-autoscaler-token-rdj28

					/registry/secrets/kube-system/job-controller-token-ztvnh

					/registry/secrets/kube-system/kube-proxy-token-zf5nj

					/registry/secrets/kube-system/namespace-controller-token-mhpzq

					/registry/secrets/kube-system/node-controller-token-t4ls4

					/registry/secrets/kube-system/persistent-volume-binder-token-js562

					/registry/secrets/kube-system/pod-garbage-collector-token-dqdhw

					/registry/secrets/kube-system/pv-protection-controller-token-p9r5b

					/registry/secrets/kube-system/pvc-protection-controller-token-j6nqp

					/registry/secrets/kube-system/replicaset-controller-token-8zsmg

					/registry/secrets/kube-system/replication-controller-token-8sjb6

					/registry/secrets/kube-system/resourcequota-controller-token-pr2sm

					/registry/secrets/kube-system/service-account-controller-token-qnmrg

					/registry/secrets/kube-system/service-controller-token-d6bz8

					/registry/secrets/kube-system/statefulset-controller-token-7zdkw

					/registry/secrets/kube-system/storage-provisioner-token-dkfxb

					/registry/secrets/kube-system/token-cleaner-token-ppr8b

					/registry/secrets/kube-system/ttl-controller-token-d5ddr

					/registry/serviceaccounts/default/default

					/registry/serviceaccounts/kube-node-lease/default

					/registry/serviceaccounts/kube-public/default

					/registry/serviceaccounts/kube-system/attachdetach-controller

					/registry/serviceaccounts/kube-system/bootstrap-signer

					/registry/serviceaccounts/kube-system/certificate-controller

					/registry/serviceaccounts/kube-system/clusterrole-aggregation-controller

					/registry/serviceaccounts/kube-system/coredns

					/registry/serviceaccounts/kube-system/cronjob-controller

					/registry/serviceaccounts/kube-system/daemon-set-controller

					/registry/serviceaccounts/kube-system/default

					/registry/serviceaccounts/kube-system/deployment-controller

					/registry/serviceaccounts/kube-system/disruption-controller

					/registry/serviceaccounts/kube-system/endpoint-controller

					/registry/serviceaccounts/kube-system/expand-controller

					/registry/serviceaccounts/kube-system/generic-garbage-collector

					/registry/serviceaccounts/kube-system/horizontal-pod-autoscaler

					/registry/serviceaccounts/kube-system/job-controller

					/registry/serviceaccounts/kube-system/kube-proxy

					/registry/serviceaccounts/kube-system/namespace-controller

					/registry/serviceaccounts/kube-system/node-controller

					/registry/serviceaccounts/kube-system/persistent-volume-binder

					/registry/serviceaccounts/kube-system/pod-garbage-collector

					/registry/serviceaccounts/kube-system/pv-protection-controller

					/registry/serviceaccounts/kube-system/pvc-protection-controller

					/registry/serviceaccounts/kube-system/replicaset-controller

					/registry/serviceaccounts/kube-system/replication-controller

					/registry/serviceaccounts/kube-system/resourcequota-controller

					/registry/serviceaccounts/kube-system/service-account-controller

					/registry/serviceaccounts/kube-system/service-controller

					/registry/serviceaccounts/kube-system/statefulset-controller

					/registry/serviceaccounts/kube-system/storage-provisioner

					/registry/serviceaccounts/kube-system/token-cleaner

					/registry/serviceaccounts/kube-system/ttl-controller

					/registry/services/endpoints/default/hello-node

					/registry/services/endpoints/default/kubernetes

					/registry/services/endpoints/kube-system/kube-controller-manager

					/registry/services/endpoints/kube-system/kube-dns

					/registry/services/endpoints/kube-system/kube-scheduler

					/registry/services/specs/default/hello-node

					/registry/services/specs/default/kubernetes

					/registry/services/specs/kube-system/kube-dns

					/registry/storageclasses/standard



