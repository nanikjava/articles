---
date: "2020-02-26"
title: "Internals of Container Networking"
author : "Nanik Tolaram (nanikjava@gmail.com)" 

---

* References:
	* https://thenewstack.io/container-networking-landscape-cni-coreos-cnm-docker/
	* https://www.edureka.co/blog/docker-networking/
	* https://blogs.igalia.com/dpino/2016/04/10/network-namespaces/
	* https://stackoverflow.com/questions/31265993/docker-networking-namespace-not-visible-in-ip-netns-list
	* https://medium.com/@Mark.io/https-medium-com-mark-io-network-setup-with-runc-containers-46b5a9cc4c5b

* There are 2 different ways network are setup and used in container world:
    * CNM (Container Network Model) or CNI (Container Network Interface)
    * CNI implementation github.com/containernetworking/plugins/
    
* CNI plugins available    
    * IPAM
        * dhcp
        * host-local
        * static
    * Main
        * bridge
        * host-device
        * windows
        * ipvlan
        * vlan
        * loopback
        * ptp
        * macvlan            
    * Meta
        * bandwidth
        * portmap
        * firewall
        * sbr
        * flannel
        * tuning
    * CNI network are implemented as plugins github.com/containernetworking/plugins. The plugins are configured by calling the cni executable and passing a configuration file outlining the network setup config like the one below
{{< highlight text >}}    
{
    "cniVersion": "0.4.0",
    "name": "openfaas-cni-bridge",
    "plugins": [
      {
        "type": "bridge",
        "bridge": "openfaas0",
        "isGateway": true,
        "ipMasq": true,
        "ipam": {
            "type": "host-local",
            "subnet": "10.62.0.0/16",
            "routes": [ { "dst": "0.0.0.0/0" } ]
               }
      },
      {
        "type": "firewall"
      }
    ]
}
{{< /highlight >}}

    reading the config file the 'bridge' plugin will be executed (which resides inside /opt/cni/bin directory).
    
    * go client implementation resides in https://github.com/containerd/go-cni the project support        
    * 

* In Linux environment network are configured and managed using [netlink](https://en.wikipedia.org/wiki/Netlink)

* containerd uses CNI
* Docker uses CNM

	
	* Docker uses CNM which uses the libnetwork project. 

* The libnetwork project uses netlink call to create network virtual connection (endpoints, etc). Docker create different namespaces required for isolation as shown in the example below

{{< highlight text >}}    
docker container list

CONTAINER ID        IMAGE               COMMAND             CREATED             STATUS              PORTS               NAMES
bbea5d31d581        ubuntu:latest       "/bin/bash"         13 seconds ago      Up 12 seconds                           peaceful_faraday
{{< /highlight >}}

{{< highlight text >}}    
docker inspect -f '{{.State.Pid}}' bbea5d31d581
19025
{{< /highlight >}}

{{< highlight text >}}    
sudo ls /proc/19025/ns/ -la
total 0
dr-x--x--x 2 root root 0 Feb 06 21:08 .
dr-xr-xr-x 9 root root 0 Feb 06 21:07 ..
lrwxrwxrwx 1 root root 0 Feb 06 21:08 cgroup -> 'cgroup:[4026531835]'
lrwxrwxrwx 1 root root 0 Feb 06 21:08 ipc -> 'ipc:[4026533665]'
lrwxrwxrwx 1 root root 0 Feb 06 21:08 mnt -> 'mnt:[4026533663]'
lrwxrwxrwx 1 root root 0 Feb 06 21:08 net -> 'net:[4026532008]'
lrwxrwxrwx 1 root root 0 Feb 06 21:08 pid -> 'pid:[4026533666]'
lrwxrwxrwx 1 root root 0 Feb 06 21:08 pid_for_children -> 'pid:[4026533666]'
lrwxrwxrwx 1 root root 0 Feb 06 21:08 user -> 'user:[4026531837]'
lrwxrwxrwx 1 root root 0 Feb 06 12:08 uts -> 'uts:[4026533664]'
{{< /highlight >}}


However running the 'ip' command does not find the namespace, as Docker does not create the symlink under the /var/run/netns folder. The 'ip' command looked under /var/run/netns for inspecting network namespaces. To allow us to use 'ip' tool we can symlink the process as follows

{{< highlight text >}}    
# (as root)
pid=$(docker inspect -f '{{.State.Pid}}' ${container_id})
mkdir -p /var/run/netns/
ln -sfT /proc/$pid/ns/net /var/run/netns/$container_id

# e.g. show stats about eth0 inside the container 
ip netns exec "${container_id}" ip -s link show eth0
{{< /highlight >}}

Another alternative is to use the 'nsenter' tool, this does not requires creating symlink:

{{< highlight text >}}
sudo nsenter -t <container_pid> -n <command>
{{< /highlight >}}

* Can use 'ip monitor' to look at the netlink activity:

{{< highlight text >}}
sudo ip monitor
{{< /highlight >}}

The following output shows the activity performed inside netlink when the command 'docker network rm  test' is executed

{{< highlight text >}}
Deleted dev br-b076a0ef8a35 lladdr 02:42:fb:1c:c1:e4 PERMANENT
Deleted dev br-b076a0ef8a35 lladdr 02:42:fb:1c:c1:e4 PERMANENT
9: br-b076a0ef8a35: <BROADCAST,MULTICAST> mtu 1500 qdisc noqueue state DOWN group default 
    link/ether 02:42:fb:1c:c1:e4 brd ff:ff:ff:ff:ff:ff
Deleted 224.0.0.22 dev br-b076a0ef8a35 lladdr 01:00:5e:00:00:16 NOARP
Deleted 224.0.0.251 dev br-b076a0ef8a35 lladdr 01:00:5e:00:00:fb NOARP
Deleted 9: br-b076a0ef8a35    inet 172.18.0.1/16 brd 172.18.255.255 scope global br-b076a0ef8a35
       valid_lft forever preferred_lft forever
Deleted local 172.18.0.1 dev br-b076a0ef8a35 table local proto kernel scope host src 172.18.0.1 
Deleted inet br-b076a0ef8a35 
Deleted inet6 br-b076a0ef8a35 
Deleted 9: br-b076a0ef8a35: <BROADCAST,MULTICAST> mtu 1500 qdisc noop state DOWN group default 
    link/ether 02:42:fb:1c:c1:e4 brd ff:ff:ff:ff:ff:ff
{{< /highlight >}}

* Docker takes care of setting up the network namespace for containers by utilizing the project github.com/vishvananda/netlink. The netlink project provides all the necessary API to setup network infrastructure via netlink

* containerd uses the same method to create network for containers, the only difference it uses a different API than Docker. The CNI API is used internally by containerd, the shim

* Tools
    * nsenter
    * ip
