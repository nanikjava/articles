---
date: "2019-09-06"
title: "Minikube Startup Process"
---

<h1>Overview</h1>

[Minikube](https://github.com/kubernetes/minikube) is called cluster-in-a-box and the reason why it is called that is because it contains everything that you will need to run cluster on a local environment. The tool is good to get a head strat into the Kubernetes world.

Kubernetes as a project is a collection of tools that works together to deliver container clustering solution. The difference with minikube is that as a developer we don't have to worry installing the different projects together as it's neatly packaged into a simple file that can be run directly.

This article will outlined the different steps that minikube goes through until it starts up and running ready to be used for deploying application. This article assume you have successfully gone through the steps outlined in the [Building and hacking minikube ](https://nanikjava.github.io/post/minikube/) article.


***NOTE: This article is using VirtualBox so there are some explanation which are VirtualBox specific***

------
<h1>Startup Flow</h1>

Following are the startup process when minikube starts up:

1. Download .iso file if it is not available locally
2. Extract boot2docker.iso from the downloaded .iso file
3. Create on-the-fly certificate to be used for ssh purposes
4. Create VirtualBox VM file with specified configuration
5. Setup storage to mount boot2docker.iso file
6. Setup network related configuration (IP, DHCP, etc)
7. Setup SSH inside the VM
8. Startup VM
9. Setup /etc/hostname, /etc/hosts 
10. Setup systemd relevant files to allow docker to start properly
11. Prepare Kubernetes and Docker 
12. Download all the relevant Kubernetes files - kubelet, kubeadm, etc
13. Pulling docker images for the different packages needed for Kubernetes
14. Start up different services such as - etcd, scheduler, controller, apiserver, etc

All configuration, keys, iso image, etc that are used to prepare minikube are available inside $HOME/.minikube. Following is how the directory structure looks like

{{< highlight html >}}
├── [4.0K]  addons
├── [1.3K]  apiserver.crt
├── [1.6K]  apiserver.key
├── [4.0K]  cache
│   ├── [4.0K]  images
│   │   ├── [4.0K]  gcr.io
│   │   │   └── [4.0K]  k8s-minikube
│   │   │       └── [ 20M]  storage-provisioner_v1.8.1
│   │   └── [4.0K]  k8s.gcr.io
│   │       ├── [ 12M]  coredns_1.3.1
│   │       ├── [ 73M]  etcd_3.3.10
│   │       ├── [ 11M]  k8s-dns-dnsmasq-nanny-amd64_1.14.13
│   │       ├── [ 14M]  k8s-dns-kube-dns-amd64_1.14.13
│   │       ├── [ 12M]  k8s-dns-sidecar-amd64_1.14.13
│   │       ├── [ 29M]  kube-addon-manager_v9.0
│   │       ├── [ 47M]  kube-apiserver_v1.15.2
│   │       ├── [ 46M]  kube-controller-manager_v1.15.2
│   │       ├── [ 29M]  kube-proxy_v1.15.2
│   │       ├── [ 43M]  kubernetes-dashboard-amd64_v1.10.1
│   │       ├── [ 28M]  kube-scheduler_v1.15.2
│   │       └── [312K]  pause_3.1
│   ├── [4.0K]  iso
│   │   └── [131M]  minikube-v1.3.0.iso
│   └── [4.0K]  v1.15.2
│       ├── [ 38M]  kubeadm
│       └── [114M]  kubelet
├── [1.0K]  ca.crt
├── [1.6K]  ca.key
├── [1.0K]  ca.pem
├── [1.0K]  cert.pem
├── [4.0K]  certs
│   ├── [1.6K]  ca-key.pem
│   ├── [1.0K]  ca.pem
│   ├── [1.0K]  cert.pem
│   └── [1.6K]  key.pem
├── [1.1K]  client.crt
├── [1.6K]  client.key
├── [4.0K]  config
├── [4.0K]  files
├── [1.6K]  key.pem
├── [  29]  last_update_check
├── [4.0K]  logs
├── [4.0K]  machines
│   ├── [4.0K]  minikube
│   │   ├── [131M]  boot2docker.iso
│   │   ├── [2.7K]  config.json
│   │   ├── [2.0G]  disk.vmdk
│   │   ├── [1.6K]  id_rsa
│   │   ├── [ 381]  id_rsa.pub
│   │   └── [4.0K]  minikube
│   │       ├── [4.0K]  Logs
│   │       │   └── [ 80K]  VBox.log
│   │       ├── [3.7K]  minikube.vbox
│   │       └── [3.6K]  minikube.vbox-prev
│   ├── [1.6K]  server-key.pem
│   └── [1.1K]  server.pem
├── [4.0K]  profiles
│   └── [4.0K]  minikube
│       └── [1.5K]  config.json
├── [1.0K]  proxy-client-ca.crt
├── [1.6K]  proxy-client-ca.key
├── [1.1K]  proxy-client.crt
└── [1.6K]  proxy-client.key
{{< /highlight >}}

--------
<h1>Inside VirtualBox</h1>

Once minikube is up and running we can ssh into the VM. Setup the configuration as shown in the screenshot below

![MinikubeSSH](/media/minikube_ssh.png)

You will get output that looks like the following and it will display a minikube shell for you to work inside the VM.	

{{< highlight html >}}

COMMAND: /usr/bin/VBoxManage showvminfo minikube --machinereadable
STDOUT:
{
name="minikube"
groups="/"
ostype="Linux 2.6 / 3.x / 4.x (64-bit)"
UUID="xxxxxx"
CfgFile="/home/nanik/.minikube/machines/minikube/minikube/minikube.vbox"
SnapFldr="/home/nanik/.minikube/machines/minikube/minikube/Snapshots"
LogFldr="/home/nanik/.minikube/machines/minikube/minikube/Logs"
	....
	....
	....
}
STDERR:
{
}
Using SSH client type: native
....                         _             _            
            _         _ ( )           ( )           
  ___ ___  (_)  ___  (_)| |/')  _   _ | |_      __  
/' _ ` _ `\| |/' _ `\| || , <  ( ) ( )| '_`\  /'__`\
| ( ) ( ) || || ( ) || || |\`\ | (_) || |_) )(  ___/
(_) (_) (_)(_)(_) (_)(_)(_) (_)`\___/'(_,__/'`\____)

$ 
{{< /highlight >}}

Executing the following ps command 

{{< highlight go >}}sudo ps -A{{< /highlight >}}

will show the different process that is currently running inside the VM. As can be seen there are a number of Kubernetes service that are running.


{{< highlight html >}}
  PID TTY          TIME CMD
    1 ?        00:00:19 systemd
    2 ?        00:00:00 kthreadd
    3 ?        00:00:01 kworker/0:0
    4 ?        00:00:00 kworker/0:0H
    6 ?        00:00:00 mm_percpu_wq
   18 ?        00:00:00 netns
   19 ?        00:00:00 kauditd
  371 ?        00:00:00 oom_reaper
  455 ?        00:00:00 writeback
  456 ?        00:00:00 kcompactd0
  458 ?        00:00:00 crypto
  459 ?        00:00:00 kblockd
  518 ?        00:00:00 ata_sff
  535 ?        00:00:00 md
	.....
 1223 ?        00:00:00 sshd
 1225 ?        00:00:00 sshd
 1226 pts/0    00:00:00 bash
 1241 ?        00:00:00 ipv6_addrconf
 1246 ?        00:00:00 ceph-msgr
 	.....
 2014 tty1     00:00:00 getty
 2018 ?        00:00:00 rpcbind
 2028 ?        00:00:03 VBoxService
 2033 ?        00:00:01 kworker/0:1H
 2059 ?        00:00:00 lockd
 2182 ?        00:00:00 sshd
  	.....
 2369 ?        00:04:42 dockerd
 2376 ?        00:02:10 containerd
 3644 ?        00:00:00 containerd-shim
 3648 ?        00:00:00 containerd-shim
 3656 ?        00:00:00 containerd-shim
 3665 ?        00:00:00 containerd-shim
 3719 ?        00:00:00 pause
 3725 ?        00:00:00 pause
 3732 ?        00:00:00 pause
 3734 ?        00:00:00 pause
 3865 ?        00:00:00 containerd-shim
  	.....
 3939 ?        00:07:46 kube-apiserver
 3948 ?        00:00:00 bash
 3970 ?        00:04:27 etcd
 3975 ?        00:00:23 kube-scheduler
 4321 ?        00:00:00 containerd-shim
 4351 ?        00:00:00 containerd-shim
 4361 ?        00:00:00 containerd-shim
  	.....
 4470 ?        00:00:00 containerd-shim
 4498 ?        00:00:12 kube-proxy
 4811 ?        00:00:00 containerd-shim
 4829 ?        00:00:47 coredns
 4864 ?        00:00:00 containerd-shim
 4882 ?        00:00:00 pause
 4906 ?        00:00:00 containerd-shim
 4925 ?        00:00:07 storage-provisi
 4996 ?        00:00:00 containerd-shim
 5013 ?        00:00:51 coredns
29243 ?        00:00:23 kubelet
29441 ?        00:00:00 containerd-shim
  	.....
29483 ?        00:00:00 containerd-shim
29499 ?        00:00:10 kube-controller
{{< /highlight >}}

Let's query docker and see what are the images that are available inside the VM

{{< highlight html >}}
$ docker image list
REPOSITORY                                TAG                 IMAGE ID            CREATED             SIZE
k8s.gcr.io/kube-scheduler                 v1.15.2             88fa9cb27bd2        4 weeks ago         81.1MB
k8s.gcr.io/kube-controller-manager        v1.15.2             9f5df470155d        4 weeks ago         159MB
k8s.gcr.io/kube-apiserver                 v1.15.2             34a53be6c9a7        4 weeks ago         207MB
k8s.gcr.io/kube-proxy                     v1.15.2             167bbf6c9338        4 weeks ago         82.4MB
k8s.gcr.io/kube-addon-manager             v9.0                119701e77cbc        7 months ago        83.1MB
k8s.gcr.io/coredns                        1.3.1               eb516548c180        7 months ago        40.3MB
k8s.gcr.io/kubernetes-dashboard-amd64     v1.10.1             f9aed6605b81        8 months ago        122MB
k8s.gcr.io/etcd                           3.3.10              2c4adeb21b4f        9 months ago        258MB
k8s.gcr.io/k8s-dns-sidecar-amd64          1.14.13             4b2e93f0133d        11 months ago       42.9MB
k8s.gcr.io/k8s-dns-kube-dns-amd64         1.14.13             55a3c5209c5e        11 months ago       51.2MB
k8s.gcr.io/k8s-dns-dnsmasq-nanny-amd64    1.14.13             6dc8ef8287d3        11 months ago       41.4MB
k8s.gcr.io/pause                          3.1                 da86e6ba6ca1        20 months ago       742kB
gcr.io/k8s-minikube/storage-provisioner   v1.8.1              4689081edb10        22 months ago       80.8MB
{{< /highlight >}}


Minikube cod uses VirtualBox's CLI command called VBoxManage, which resides inside /usr/bin. Following log snippets show some of the call that are performed when minikube starts up

{{< highlight html >}}
	......
COMMAND: /usr/bin/VBoxManage --version
	......

COMMAND: /usr/bin/VBoxManage list hostonlyifs
	......

Downloading /home/nanik/.minikube/cache/boot2docker.iso from file:///home/nanik/.minikube/cache/iso/minikube-v1.3.0.iso...
Creating VirtualBox VM...
Creating SSH key...
Creating disk image...
Creating 20000 MB hard disk image...
Writing magic tar header
Writing SSH key tar header
Calling inner createDiskImage
&{/usr/bin/VBoxManage [/usr/bin/VBoxManage convertfromraw stdin /home/nanik/.minikube/machines/minikube/disk.vmdk 20971520000 --format VMDK] [] 	......}
Starting command
Copying to stdin
Filling zeroes
Closing STDIN
Waiting on cmd
COMMAND: /usr/bin/VBoxManage createvm --basefolder /home/nanik/.minikube/machines/minikube --name minikube --register
	......

COMMAND: /usr/bin/VBoxManage modifyvm minikube --firmware bios --bioslogofadein off --bioslogofadeout off --bioslogodisplaytime 0 --biosbootmenu disabled --ostype Linux26_64 --cpus 2 --memory 2000 --acpi on --ioapic on --rtcuseutc on --natdnshostresolver1 on --natdnsproxy1 off --cpuhotplug off --pae on --hpet on --hwvirtex on --nestedpaging on --largepages on --vtxvpid on --accelerate3d off --boot1 dvd
	......

COMMAND: /usr/bin/VBoxManage modifyvm minikube --nic1 nat --nictype1 virtio --cableconnected1 on
	......

COMMAND: /usr/bin/VBoxManage storagectl minikube --name SATA --add sata --hostiocache on
	......

COMMAND: /usr/bin/VBoxManage storageattach minikube --storagectl SATA --port 0 --device 0 --type dvddrive --medium /home/nanik/.minikube/machines/minikube/boot2docker.iso
	......

COMMAND: /usr/bin/VBoxManage storageattach minikube --storagectl SATA --port 1 --device 0 --type hdd --medium /home/nanik/.minikube/machines/minikube/disk.vmdk
	......

COMMAND: /usr/bin/VBoxManage guestproperty set minikube /VirtualBox/GuestAdd/SharedFolders/MountPrefix /
	......

COMMAND: /usr/bin/VBoxManage guestproperty set minikube /VirtualBox/GuestAdd/SharedFolders/MountDir /
	......

setting up shareDir '/home' -> 'hosthome'
COMMAND: /usr/bin/VBoxManage sharedfolder add minikube --name hosthome --hostpath /home --automount
	......

COMMAND: /usr/bin/VBoxManage setextradata minikube VBoxInternal2/SharedFoldersEnableSymlinksCreate/hosthome 1
	......

Starting the VM...
COMMAND: /usr/bin/VBoxManage showvminfo minikube --machinereadable
	......

Check network to re-create if needed...
COMMAND: /usr/bin/VBoxManage list hostonlyifs
	......

Searching for hostonly interface for IPv4: xxxxx and Mask: xxxxx
Found: vboxnet0
COMMAND: /usr/bin/VBoxManage list dhcpservers
	......

Removing orphan DHCP servers...
COMMAND: /usr/bin/VBoxManage list hostonlyifs
	......

Adding/Modifying DHCP server "xxxxxxx" with address range "xxxx" - "yyyyy"...
COMMAND: /usr/bin/VBoxManage list dhcpservers
	......


{{< /highlight >}}
