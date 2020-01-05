---
date: "2019-11-30"
title: "Kubernetes Brain Dump (ALWAYS WIP)"
author : "Nanik Tolaram (nanikjava@gmail.com)" 

---
* Linux Network Namespaces is one of the important feature in Linux Kernel that are leveraged by containers like docker and kubernetes. It is used in the CNI (Container Network Interface).
* A Kubernetes cluster can be deployed on either physical or virtual machines.
* Once you've created a Deployment, the Kubernetes master schedules mentioned application instances onto individual Nodes in the cluster.
* Once the application instances are created, a Kubernetes Deployment Controller continuously monitors those instances. If the Node hosting an instance goes down or is deleted, the Deployment controller replaces the instance with an instance on another Node in the cluster. This provides a self-healing mechanism to address machine failure or maintenance.
* **Node** - A node is a VM or a physical computer that serves as a worker machine in a Kubernetes cluster. 
* Each node has a Kubelet, which is an agent for managing the node and communicating with the Kubernetes master. The node should also have tools for handling container operations, such as Docker or rkt.
* A Kubernetes cluster that handles production traffic should have a minimum of three nodes.
* The nodes communicate with the master using the Kubernetes API. End users can also use the Kubernetes API directly to interact with the cluster.    
* Node Components run on every node, maintaining running pods and providing the Kubernetes runtime environment.

    * **kubelet** - An agent that runs on each node in the cluster. It makes sure that containers are running in a pod.  The kubelet takes a set of PodSpecs that are provided through various mechanisms and ensures that the containers described in those PodSpecs are running and healthy. The kubelet doesn’t manage containers which were not created by Kubernetes.

    * **kube-proxy** - kube-proxy is a network proxy that runs on each node in your cluster, implementing part of the Kubernetes Service concept. kube-proxy maintains network rules on nodes. These network rules allow network communication to your Pods from network sessions inside or outside of your cluster. kube-proxy uses the operating system packet filtering layer if there is one and it’s available. Otherwise, kube-proxy forwards the traffic itself.

    * **Container Runtime** - The container runtime is the software that is responsible for running containers.
    
    - A node is a worker machine in Kubernetes, previously known as a minion. A node may be a VM or physical machine, depending on the cluster. Each node contains the services necessary to run pods and is managed by the master components.
    
    - A node’s status contains the following information:
        * Addresses
        * Conditions
        * Capacity and Allocatable
        * Info

<h2>Pod</h2>
    
* **Pod** is something logical and not physical. It is just a logical wrapper for a set of containers with it's resources.  In terms of Docker constructs, a Pod is modelled as a group of Docker containers with shared namespaces and shared filesystem volumes.
* Containers within a Pod share an IP address and port space, and can find each other via localhost. They can also communicate with each other using standard inter-process communications like SystemV semaphores or POSIX shared memory. Containers in different Pods have distinct IP addresses and can not communicate by IPC without special configuration. These containers usually communicate with each other via Pod IP addresses.
* A Pod is a group of one or more containers (such as Docker containers), with shared storage/network, and a specification for how to run the containers.
* A Pod is a Kubernetes abstraction that represents a group of one or more application containers (such as _docker_ or _rkt_), and some shared resources for those containers. Those resources include:
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
        * Kubelet, a process responsible for communication between the Kubernetes Master and the Node; it manages the Pods and the containers running on a machine.
        * A container runtime (like Docker, rkt) responsible for pulling the container image from a registry, unpacking the container, and running the application.
        * Containers should only be scheduled together in a single Pod if they are tightly coupled and need to share resources such as disk.
    
            ![ContainersPods](https://d33wubrfki0l68.cloudfront.net/5cb72d407cbe2755e581b6de757e0d81760d5b86/a9df9/docs/tutorials/kubernetes-basics/public/images/module_03_nodes.svg)

* The Kubernetes Master is a collection of three processes that run on a single node in your cluster, which is designated as the master node. Those processes are: kube-apiserver, kube-controller-manager and kube-scheduler.
    - Each individual non-master node in your cluster runs two processes:
        - kubelet, which communicates with the Kubernetes Master.
        - kube-proxy, a network proxy which reflects Kubernetes networking services on each node.

<h2>Services</h2>

- A Service in Kubernetes is an abstraction which defines a logical set of Pods and a policy by which to access them. Services enable a loose coupling between dependent Pods. A Service is defined using YAML (preferred) or JSON, like all Kubernetes objects. The set of Pods targeted by a Service is usually determined by a LabelSelector (see below for why you might want a Service without including selector in the spec).
- Services can be exposed in different ways by specifying a type in the ServiceSpec:
    - **ClusterIP** (default) - Exposes the Service on an internal IP in the cluster. This type makes the Service only reachable from within the cluster.
    - **NodePort** - Exposes the Service on the same port of each selected Node in the cluster using NAT. Makes a Service accessible from outside the cluster using <NodeIP>:<NodePort>. Superset of ClusterIP.
    - **LoadBalancer** - Creates an external load balancer in the current cloud (if supported) and assigns a fixed, external IP to the Service. Superset of NodePort.
    - **ExternalName** - Exposes the Service using an arbitrary name (specified by externalName in the spec) by returning a CNAME record with the name. No proxy is used. This type requires v1.7 or higher of kube-dns.

- Services match a set of Pods using labels and selectors, a grouping primitive that allows logical operation on objects in Kubernetes. Labels are key/value pairs attached to objects and can be used in any number of ways:

    * Designate objects for development, test, and production
    * Embed version tags
    * Classify an object using tags

    ![Services](https://d33wubrfki0l68.cloudfront.net/cc38b0f3c0fd94e66495e3a4198f2096cdecd3d5/ace10/docs/tutorials/kubernetes-basics/public/images/module_04_services.svg)
    ![Services](https://d33wubrfki0l68.cloudfront.net/b964c59cdc1979dd4e1904c25f43745564ef6bee/f3351/docs/tutorials/kubernetes-basics/public/images/module_04_labels.svg)

* Once you’ve set your desired state, the Kubernetes Control Plane makes the cluster’s current state match the desired state via the Pod Lifecycle Event Generator (PLEG). The Kubernetes Control Plane consists of a collection of processes running on your cluster:
    - The various parts of the Kubernetes Control Plane, such as the Kubernetes Master and kubelet processes, govern how Kubernetes communicates with your cluster. The Control Plane maintains a record of all of the Kubernetes Objects in the system, and runs continuous control loops to manage those objects’ state. At any given time, the Control Plane’s control loops will respond to changes in the cluster and work to make the actual state of all the objects in the system match the desired state that you provided.

* Master Components
    -Master components provide the cluster’s control plane. Master components make global decisions about the cluster (for example, scheduling), and they detect and respond to cluster events (for example, starting up a new pod when a deployment’s replicas field is unsatisfied).  For simplicity, set up scripts typically start all master components on the same machine, and do not run user containers on this machine. See Building High-Availability Clusters for an example multi-master-VM setup.

    * **kube-apiserver** - Component on the master that exposes the Kubernetes API. It is the front-end for the Kubernetes control plane.

    * **etcd** - Consistent and highly-available key value store used as Kubernetes’ backing store for all cluster data.

    * **kube-scheduler** - Component on the master that watches newly created pods that have no node assigned, and selects a node for them to run on.

    * **kube-controller-manager** - Component on the master that runs controllers.
        
          ![KubernetesControllerManager](https://d33wubrfki0l68.cloudfront.net/e298a92e2454520dddefc3b4df28ad68f9b91c6f/70d52/images/docs/pre-ccm-arch.png)
      
    
    * Logically, each controller is a separate process, but to reduce complexity, they are all compiled into a single binary and run in a single process. These controllers include:
        - Node Controller: Responsible for noticing and responding when nodes go down.
        - Replication Controller: Responsible for maintaining the correct number of pods for every replication controller object in the system.
        - Endpoints Controller: Populates the Endpoints object (that is, joins Services & Pods).
        - Service Account & Token Controllers: Create default accounts and API access tokens for new namespaces.

    * **cloud-controller-manager** - cloud-controller-manager runs controllers that interact with the underlying cloud providers. The cloud-controller-manager binary is an alpha feature introduced in Kubernetes release 1.6.

        - cloud-controller-manager runs cloud-provider-specific controller loops only. You must disable these controller loops in the kube-controller-manager. You can disable the controller loops by setting the --cloud-provider flag to external when starting the kube-controller-manager. cloud-controller-manager allows the cloud vendor’s code and the Kubernetes code to evolve independently of each other. In prior releases, the core Kubernetes code was dependent upon cloud-provider-specific code for functionality. In future releases, code specific to cloud vendors should be maintained by the cloud vendor themselves, and linked to cloud-controller-manager while running Kubernetes.

        - The following controllers have cloud provider dependencies:

            [x] **Node Controller** -- For checking the cloud provider to determine if a node has been deleted in the cloud after it stops responding
            
            [x] **Route Controller** -- For setting up routes in the underlying cloud infrastructure
            
            [x] **Service Controller** - For creating, updating and deleting cloud provider load balancers
            
            [x] **Volume Controller** - For creating, attaching, and mounting volumes, and interacting with the cloud provider to orchestrate volumes
                
    ![KubernetesControllerManager](https://d33wubrfki0l68.cloudfront.net/518e18713c865fe67a5f23fc64260806d72b38f5/61d75/images/docs/post-ccm-arch.png)


<h2>Addons</h2>

Addons use Kubernetes resources (DaemonSet, Deployment, etc) to implement cluster features. Because these are providing cluster-level features, namespaced resources for addons belong within the kube-system namespace.

* **DNS** - While the other addons are not strictly required, all Kubernetes clusters should have cluster DNS, as many examples rely on it. Cluster DNS is a DNS server, in addition to the other DNS server(s) in your environment, which serves DNS records for Kubernetes services. Containers started by Kubernetes automatically include this DNS server in their DNS searches.

* **Web UI (Dashboard)** - Dashboard is a general purpose, web-based UI for Kubernetes clusters. It allows users to manage and troubleshoot applications running in the cluster, as well as the cluster itself.

* **Container Resource Monitoring** - Container Resource Monitoring records generic time-series metrics about containers in a central database, and provides a UI for browsing that data.

* **Cluster-level Logging** - Cluster-level logging mechanism is responsible for saving container logs to a central log store with search/browsing interface.

* The basic Kubernetes objects include:
    - **Pod**
    - **Service**
    - **Volume**
    - **Namespace**

<h2>kubeadm</h2>

* Running 'kubeadm init' will run phases of initialization to setup Kubernetes cluster. The code that break down these steps reside inside app/cmd/phases/init
    * preflight step reside inside app/preflight
    * **app/preflight/checks.go** -- contains lots of checking related to preflight (check port availability, check existence of directory, check app availability in PATH, etc)
    * **app/cmd/phases** -- these directory contains code that is tied to the different command line function in kubeadm (eg: phases/init --> has code that link to 'kubeadm init'
        * **/init** -- directory contains code for all the phases when executing 'kubeadm init'
        * **/join** -- directory contains code for the 'kubeadm join' command
        * **/reset** -- directory contains code for the 'kubeadm reset' command
        * **/upgrade** -- directory contains code for the 'kubeadm upgrade' command
    * **app/phases** -- contains code used as part of the kubeadm phases 
    * **app/util/system**
        * **cgroup_validator** -- validator for cgroups
        * **docker_validator** -- docker validator
        * **kernel_validator** -- kernel validator
        * **os_validator** -- OS validator (version)
        * **package_validator** -- package manager validator


<h2>CSI (Container Storage Interface)</h2> 

* Interfacing K8 with storage implementation		
    * **Node** --  A host where a workload (such as Pods in Kubernetes) will be running.
    * **Plugin** - In the CSI world, this points to a service that exposes gRPC endpoints
      ![CSIPicture](https://arslan.io/images/how-to-write-a-container-storage-interface-%28csi%29-plugin-2.jpg) 
* Specification defines the boundary between the CO and the CSI plugin. 
* Plugin part actually is separated into two individual plugins.
    * **Node Plugin**
        * The Node plugin is a gRPC server that needs to run on the Node where the volume will be provisioned. So suppose you have a Kubernetes cluster with three nodes where your Pod’s are scheduled, you would deploy this to all three nodes.
    * **Controller Plugin**
        * The Controller plugin is a gRPC server that can run anywhere. In terms of a Kubernetes cluster, it can run on any node (even on the master node).
  These two entities can live in a single binary or you can separate them.
     ![CSIPicture](https://arslan.io/images/how-to-write-a-container-storage-interface-%28csi%29-plugin-3.jpg) 

<h2>References</h2>

* Good example and explanation on how to implement CSI -- https://arslan.io/2018/06/21/how-to-write-a-container-storage-interface-csi-plugin/
* https://www.youtube.com/watch?v=_qfSzrPn9Cs



<h2>Experimenting with Kubernetes</h2>

* A good guide on cloud provider 					        -- https://github.com/hobby-kube/guide
* Kubernetes on ARM (ODROID, RPi, Upboard combo) 			-- https://github.com/luxas/kubernetes-on-arm
* Kubernetes on ODROID N2 						            -- https://www.trion.de/news/2019/05/06/kubernetes-odroid-n2.html
* Setting up a Multi-Arch, Multi OS cluster 			    -- https://gist.github.com/PatrickLang/28a05cfd6cf4322d519d04cff0996585
* Kubernetes on Raspberry Pi: Challenges & Advantages 		-- https://www.weave.works/blog/kubernetes-raspberry-pi/
* Intalling kubernetes using kubeadm 				        -- https://www.mirantis.com/blog/how-install-kubernetes-kubeadm/
* Kubernetes HA control plane 					            -- https://octetz.com/posts/ha-control-plane-k8s-kubeadm


<h2>Books</h2>

* Programming Kubernetes - OReilly
* Kubernetes Patterns    - OReilly
* Managing Kubernetes    - OReilly


<h2>References</h2>

- https://blogs.igalia.com/dpino/2016/04/10/network-namespaces/
- https://github.com/containernetworking/cni/blob/master/SPEC.md
- https://github.com/containernetworking/plugins
- https://medium.com/@vikram.fugro/container-networking-interface-aka-cni-bdfe23f865cf --> show how the CNI works using the containetworking github.com account
- https://www.cncf.io/blog/2017/05/23/cncf-hosts-container-networking-interface-cni/ --> specification and implementation of CNI
- https://www.weave.works/docs/net/latest/kubernetes/kube-addon/ --> weavenet is network plugin for docker for variety of things (service discovery, etc), this website outlined how to use it as inside kubernetes 
- [Good explanation about how the different parts of Kubernetes works](https://github.com/darshanime/notes/blob/master/kubernetes.org)
