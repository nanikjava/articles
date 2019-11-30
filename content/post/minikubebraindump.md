---
date: "2019-09-01"
title: "Minikube Brain Dump (ALWAYS WIP)"
author : "Nanik Tolaram (nanikjava@gmail.com)" 

---


* ETCD inside minikube
	- To communicate to etcd that are running inside minikube we need to do the following
{{< highlight bash >}}
minikube ssh
{{< /highlight >}}
{{< highlight bash >}}
/hosthome/nanik/Downloads/temp/packages/src/go.etcd.io/etcd/etcdctl/etcdctl --cacert=/var/lib/minikube/certs/etcd/ca.crt --key=/var/lib/minikube/certs/etcd/ca.key --cert=/var/lib/minikube/certs/etcd/ca.crt get  --prefix=true ""--> the ca.crt and ca.key resides inside minikube VM
{{< /highlight >}}
{{< highlight bash >}}    
ETCDCTL_API=3  /hosthome/nanik/Downloads/temp/packages/src/go.etcd.io/etcd/etcdctl/etcdctl --cacert=/var/lib/minikube/certs/etcd/ca.crt --key=/var/lib/minikube/certs/etcd/ca.key --cert=/var/lib/minikube/certs/etcd/ca.crt get  --from-key '' --keys-only --> will get all the keys stored inside
{{< /highlight >}}
**/hostname** -- is the default mount volume created by minikube to access host directory.        


        >/registry/apiregistration.k8s.io/apiservices/v1
        
        >/apiregistration.k8s.io/apiservices/v1.admissionregistration.k8s.io
        
        >/apiregistration.k8s.io/apiservices/v1.apiextensions.k8s.io
        
        >/apiregistration.k8s.io/apiservices/v1.apps
        
        >/apiregistration.k8s.io/apiservices/v1.authentication.k8s.io


					
* Minikube
	* Minikube supports lxc. LXC has it's own Go bindings called libvirt-go
	* Internally libvirt-go provides Go binding for different containers as virtualbox, kvm, etc
	* Minikube uses different way to interact with different containers as outlined below:
		- **virtualbox** -- Calling executeable /usr/bin/VboxManage
		- **kvm2**       -- it is calling docker-machine-driver-kvm2 which is part of minikube, can be found under cmd/drivers/kvm/main.go. The code is using the libvirt to communicate with kvm
		- **gvisor** -- there is some sort of dependencies with containerd and rpc-statd.service as this both service is restarted when gvisor has been download and successfully copy over (https://github.com/kubernetes/minikube/blob/master/deploy/addons/gvisor/README.md)
		- .... and others
        

* Running using the following command

    start --vm-driver=kvm2 --container-runtime=cri-o --v=8
    
 running virt-manager and enter the container and execute 'systemctl status' yield the following output
 
 
 systemctl status
● minikube
    State: degraded
     Jobs: 0 queued
   Failed: 1 units
    Since: Mon 2019-10-28 21:14:47 UTC; 4min 25s ago
   CGroup: /
           ├─init.scope
           │ └─1 /sbin/init noembed norestore
           ├─kubepods
           │ ├─besteffort
           │ │ ├─pod66fd795f-401c-4181-9cf0-0c2a17663b81
           │ │ │ ├─crio-conmon-cb4126fb3d539269743b654879485e5fa5f1d863896783a307fb43f488d29ac5
           │ │ │ │ └─3584 /usr/libexec/crio/conmon --syslog -c cb4126fb3d539269743b654879485e5fa5f1d863896783a307fb43f488d29ac5 -n k8s_POD_kube-proxy-plfd2_kube-system_66fd795f-401c-4181-9cf0-0c2a17663b81_0 -u cb4126fb3d539269743b654879485e5fa5f1d863896783a307fb43f488d29ac5 -r /usr/bin/runc -b /var/run/containers/storage/overlay-containers/cb4126fb3d539269743b654879485e5fa5f1d863896783a307fb43f488d29ac5/userdata -p /var/run/containers/storage/overlay-containers/cb4126fb3d539269743b654879485e5fa5f1d863896783a307fb43f488d29ac5/userdata/pidfile -l /var/log/pods/kube-system_kube-proxy-plfd2_66fd795f-401c-4181-9cf0-0c2a17663b81/cb4126fb3d539269743b654879485e5fa5f1d863896783a307fb43f488d29ac5.log --exit-dir me-arg --root=/run/runc --no-pivotath /var/run/crio --log-level debug --runti--More-- 
           │ │ │ ├─crio-cb4126fb3d539269743b654879485e5fa5f1d863896783a307fb43f488d29ac5
           │ │ │ │ └─3598 /pause
           │ │ │ ├─crio-conmon-f25ed7b9fb31818f0b8ab6d4cb94baead72a124b8915d92145bbb5ce86811ee2
           │ │ │ │ └─3616 /usr/libexec/crio/conmon --syslog -c f25ed7b9fb31818f0b8ab6d4cb94baead72a124b8915d92145bbb5ce86811ee2 -n k8s_kube-proxy_kube-proxy-plfd2_kube-system_66fd795f-401c-4181-9cf0-0c2a17663b81_0 -u f25ed7b9fb31818f0b8ab6d4cb94baead72a124b8915d92145bbb5ce86811ee2 -r /usr/bin/runc -b /var/run/containers/storage/overlay-containers/f25ed7b9fb31818f0b8ab6d4cb94baead72a124b8915d92145bbb5ce86811ee2/userdata -p /var/run/containers/storage/overlay-containers/f25ed7b9fb31818f0b8ab6d4cb94baead72a124b8915d92145bbb5ce86811ee2/userdata/pidfile -l /var/log/pods/kube-system_kube-proxy-plfd2_66fd795f-401c-4181-9cf0-0c2a17663b81/kube-proxy/0.log --exit-dir /var/run/crio/exits --socket-dir-path /var/run/crio --log-level debug --runtime-arg --root=/run/runc --no-pivot
           │ │ │ └─crio-f25ed7b9fb31818f0b8ab6d4cb94baead72a124b8915d92145bbb5ce86811ee2
           │ │ │   └─3626 /usr/local/bin/kube-proxy --config=/var/lib/kube-proxy/config.conf --hostname-override=minikube
           │ │ ├─pod89ae5909ffeb743f1dfb204f25a829e6
           │ │ │ ├─crio-conmon-fa20492d16fce683bd7eeeef02b18abdcd46d9418036f70d9cb0764f60178610
           │ │ │ │ └─3079 /usr/libexec/crio/conmon --syslog -c fa20492d16fce683bd7eeeef02b18abdcd46d9418036f70d9cb0764f60178610 -n k8s_POD_etcd-minikube_kube-system_89ae5909ffeb743f1dfb204f25a829e6_0 -u fa20492d16fce683bd7eeeef02b18abdcd46d9418036f70d9cb0764f60178610 -r /usr/bin/runc -b /var/run/containers/storage/overlay-containers/fa20492d16fce683bd7eeeef02b18abdcd46d9418036f70d9cb0764f60178610/userdata -p /var/run/containers/storage/overlay-containers/fa20492d16fce683bd7eeeef02b18abdcd46d9418036f70d9cb0764f60178610/userdata/pidfile -l /var/log/pods/kube-system_etcd-minikube_89ae5909ffeb743f1dfb204f25a829e6/fa20492d16fce683bd7eeeef02b18abdcd46d9418036f70d9cb0764f60178610.log --exit-dir /var/run/crio/exits --socket-dir-path /var/run/crio --log-level debug --runtime-arg --root=/run/runc --no-pivot
           │ │ │ ├─crio-conmon-42866e6955f0f3d8bde90c9235c1da99498469db636e6a8404208473c2fca3d9
           │ │ │ │ └─3273 /usr/libexec/crio/conmon --syslog -c 42866e6955f0f3d8bde90c9235c1da99498469db636e6a8404208473c2fca3d9 -n k8s_etcd_etcd-minikube_kube-system_89ae5909ffeb743f1dfb204f25a829e6_0 -u 42866e6955f0f3d8bde90c9235c1da99498469db636e6a8404208473c2fca3d9 -r /usr/bin/runc -b /var/run/containers/storage/overlay-containers/42866e6955f0f3d8bde90c9235c1da99498469db636e6a8404208473c2fca3d9/userdata -p /var/run/containers/storage/overlay-containers/42866e6955f0f3d8bde90c9235c1da99498469db636e6a8404208473c2fca3d9/userdata/pidfile -l /var/log/pods/kube-system_etcd-minikube_89ae5909ffeb743f1dfb204f25a829e6/etcd/0.log --exit-dir /var/run/crio/exits --socket-dir-path /var/run/crio --log-level debug --runtime-arg --root=/run/runc --no-pivot
           │ │ │ ├─crio-fa20492d16fce683bd7eeeef02b18abdcd46d9418036f70d9cb0764f60178610
           │ │ │ │ └─3136 /pause
           │ │ │ └─crio-42866e6955f0f3d8bde90c9235c1da99498469db636e6a8404208473c2fca3d9
           │ │ │   └─3327 etcd --advertise-client-urls=https://192.168.39.214:2379 --cert-file=/var/lib/minikube/certs/etcd/server.crt --client-cert-auth=true --data-dir=/var/lib/minikube/etcd --initial-advertise-peer-urls=https://192.168.39.214:2380 --initial-cluster=minikube=https://192.168.39.214:2380 --key-file=/var/lib/minikube/certs/etcd/server.key --listen-client-urls=https://127.0.0.1:2379,https://192.168.39.214:2379 --listen-metrics-urls=http://127.0.0.1:2381 --listen-peer-urls=https://192.168.39.214:2380 --name=minikube --peer-cert-file=/var/lib/minikube/certs/etcd/peer.crt --peer-client-cert-auth=true --peer-key-file=/var/lib/minikube/certs/etcd/peer.key --peer-trusted-ca-file=/var/lib/minikube/certs/etcd/ca.crt --snapshot-count=10000 --trusted-ca-file=/var/lib/minikube/certs/etcd/ca.crt
           │ │ └─pod9a1966c7-8565-4aed-8609-04abe6bdf785
           │ │   ├─crio-21627aa0b547296c821ae7432cb4ee4ed22f2a9707c833fc6e3563d8faaf6dd9
           │ │   │ └─3870 /storage-provisioner
           │ │   ├─crio-conmon-ba42f9d76d11545bedcf8b43f19877145bb27efbfee6d11df061b8373bfe61a9
76d11545bedcf8b43f19877145bb27efbfee6d11df061b8373bfe61a9 -n k8s_POD_storage-provisioner_kube-system_9a1966c7-8565-4aed-8609-04abe6bdf785_0 -u ba42f9d76d11545bedcf8b43f19877145bb27efbfee6d11df061b8373bfe61a9 -r /usr/bin/runc -b /var/run/containers/storage/overlay-containers/ba42f9d76d11545bedcf8b43f19877145bb27efbfee6d11df061b8373bfe61a9/userdata -p /var/run/containers/storage/overlay-containers/ba42f9d76d11545bedcf8b43f19877145bb27efbfee6d11df061b8373bfe61a9/userdata/pidfile -l /var/log/pods/kube-system_storage-provisioner_9a1966c7-8565-4aed-8609-04abe6bdf785/ba42f9d76d11545bedcf8b43f19877145bb27efbfee6d11df061b8373bfe61a9.log --exit-dir /var/run/crio/exits --socket-dir-path /var/run/crio --log-level debug --runtime-arg --root=/run/runc --no-pivot
           │ │   ├─crio-ba42f9d76d11545bedcf8b43f19877145bb27efbfee6d11df061b8373bfe61a9
           │ │   │ └─3840 /pause
           │ │   └─crio-conmon-21627aa0b547296c821ae7432cb4ee4ed22f2a9707c833fc6e3563d8faaf6dd9
           │ │     └─3859 /usr/libexec/crio/conmon --syslog -c 21627aa0b547296c821ae7432cb4ee4ed22f2a9707c833fc6e3563d8faaf6dd9 -n k8s_storage-provisioner_storage-provisioner_kube-system_9a1966c7-8565-4aed-8609-04abe6bdf785_0 -u 21627aa0b547296c821ae7432cb4ee4ed22f2a9707c833fc6e3563d8faaf6dd9 -r /usr/bin/runc -b /var/run/containers/storage/overlay-containers/21627aa0b547296c821ae7432cb4ee4ed22f2a9707c833fc6e3563d8faaf6dd9/userdata -p /var/run/containers/storage/overlay-ed-8609-04abe6bdf785/storage-provisioner/0.log --exit-dir /var/run/crio/exits --socket-dir-path /var/run/crio --log-level debug --runtime-arg --root=/run/runc --no-pivot
           │ └─burstable
           │   ├─pod67888a6f41348f1a41e319a7f77279a2
           │   │ ├─crio-90f7b48799d7f40bf2f81ba9946e74f15afd7348a515ca00b4367b1e208621af
           │   │ │ └─3267 kube-controller-manager --authentication-kubeconfig=/etc/kubernetes/controller-manager.conf --authorization-kubeconfig=/etc/kubernetes/controller-manager.conf --bind-address=127.0.0.1 --client-ca-file=/var/lib/minikube/certs/ca.crt --cluster-signing-cert-file=/var/lib/minikube/certs/ca.crt --cluster-signing-key-file=/var/lib/minikube/certs/ca.key --controllers=*,bootstrapsigner,tokencleaner --kubeconfig=/etc/kubernetes/controller-manager.conf --leader-elect=true --requestheader-client-ca-file=/var/lib/minikube/certs/front-proxy-ca.crt --root-ca-file=/var/lib/minikube/certs/ca.crt --service-account-private-key-file=/var/lib/minikube/certs/sa.key --use-service-account-credentials=true
           │   │ ├─crio-conmon-027e7f3d792498b1e54bfb1b6148c4959d2d366a8af64e3ae5d0ac2aba9679df
           │   │ │ └─3096 /usr/libexec/crio/conmon --syslog -c 027e7f3d792498b1e54bfb1b6148c4959d2d366a8af64e3ae5d0ac2aba9679df -n k8s_POD_kube-controller-manager-minikube_kube-system_67888a6f41348f1a41e319a7f77279a2_0 -u 027e7f3d792498br/run/containers/storage/overlay-containers/027e7f3d792498b1e54bfb1b6148c4959d2d366a8af64e3ae5d0ac2aba9679df/userdata -p /var/run/containers/storage/overlay-containers/027e7f3d792498b1e54bfb1b6148c4959d2d366a8af64e3ae5d0ac2aba9679df/userdata/pidfile -l /var/log/pods/kube-system_kube-controller-manager-minikube_67888a6f41348f1a41e319a7f77279a2/027e7f3d792498b1e54bfb1b6148c4959d2d366a8af64e3ae5d0ac2aba9679df.log --exit-dir /var/run/crio/exits --socket-dir-path /var/run/crio --log-level debug --runtime-arg --root=/run/runc --no-pivot
           │   │ ├─crio-conmon-90f7b48799d7f40bf2f81ba9946e74f15afd7348a515ca00b4367b1e208621af
           │   │ │ └─3249 /usr/libexec/crio/conmon --syslog -c 90f7b48799d7f40bf2f81ba9946e74f15afd7348a515ca00b4367b1e208621af -n k8s_kube-controller-manager_kube-controller-manager-minikube_kube-system_67888a6f41348f1a41e319a7f77279a2_0 -u 90f7b48799d7f40bf2f81ba9946e74f15afd7348a515ca00b4367b1e208621af -r /usr/bin/runc -b /var/run/containers/storage/overlay-containers/90f7b48799d7f40bf2f81ba9946e74f15afd7348a515ca00b4367b1e208621af/userdata -p /var/run/containers/storage/overlay-containers/90f7b48799d7f40bf2f81ba9946e74f15afd7348a515ca00b4367b1e208621af/userdata/pidfile -l /var/log/pods/kube-system_kube-controller-manager-minikube_67888a6f41348f1a41e319a7f77279a2/kube-controller-manager/0.log --exit-dir /var/run/crio/exits --socket-dir-path /var/run/crio --log-level debug --runtime-arg --root=/run/runc --no-pivot
           │   │ └─crio-027e7f3d792498b1e54bfb1b6148c4959d2d366a8af64e3ae5d0ac2aba9679df
           │   │   └─3158 /pause
           │   ├─pod288cbfbd5565f92cc63b18bcafb084d5
           │   │ ├─crio-conmon-8fedc929a6c2f817e0c7855d588f6aa6a0db7e0bd5df4a7e3baa24ea8ccd0851
           │   │ │ └─3217 /usr/libexec/crio/conmon --syslog -c 8fedc929a6c2f817e0c7855d588f6aa6a0db7e0bd5df4a7e3baa24ea8ccd0851 -n k8s_kube-apiserver_kube-apiserver-minikube_kube-system_288cbfbd5565f92cc63b18bcafb084d5_0 -u 8fedc929a6c2f817e0c7855d588f6aa6a0db7e0bd5df4a7e3baa24ea8ccd0851 -r /usr/bin/runc -b /var/run/containers/storage/overlay-containers/8fedc929a6c2f817e0c7855d588f6aa6a0db7e0bd5df4a7e3baa24ea8ccd0851/userdata -p /var/run/containers/storage/overlay-containers/8fedc929a6c2f817e0c7855d588f6aa6a0db7e0bd5df4a7e3baa24ea8ccd0851/userdata/pidfile -l /var/log/pods/kube-system_kube-apiserver-minikube_288cbfbd5565f92cc63b18bcafb084d5/kube-apiserver/0.log --exit-dir /var/run/crio/exits --socket-dir-path /var/run/crio --log-level debug --runtime-arg --root=/run/runc --no-pivot
           │   │ ├─crio-8fedc929a6c2f817e0c7855d588f6aa6a0db7e0bd5df4a7e3baa24ea8ccd0851
           │   │ │ └─3228 kube-apiserver --advertise-address=192.168.39.214 --allow-privileged=true --authorization-mode=Node,RBAC --client-ca-file=/var/lib/minikube/certs/ca.crt --enable-admission-plugins=NamespaceLifecycle,LimitRanger,ServiceAccount,DefaultStorageClass,DefaultTolerationSeconds,NodeRestriction,MutatingAdmissionWebhook,ValidatingAdmissionWebhook,ResourceQuota --enable-bootstrap-token-auth=true --etcd-cafile=/var/lib/minikube/certs/etcd/ca.crt --etcd-cerlib/minikube/certs/apiserver-etcd-client.key --etcd-servers=https://127.0.0.1:2379 --insecure-port=0 --kubelet-client-certificate=/var/lib/minikube/certs/apiserver-kubelet-client.crt --kubelet-client-key=/var/lib/minikube/certs/apiserver-kubelet-client.key --kubelet-preferred-address-types=InternalIP,ExternalIP,Hostname --proxy-client-cert-file=/var/lib/minikube/certs/front-proxy-client.crt --proxy-client-key-file=/var/lib/minikube/certs/front-proxy-client.key --requestheader-allowed-names=front-proxy-client --requestheader-client-ca-file=/var/lib/minikube/certs/front-proxy-ca.crt --requestheader-extra-headers-prefix=X-Remote-Extra- --requestheader-group-headers=X-Remote-Group --requestheader-username-headers=X-Remote-User --secure-port=8443 --service-account-key-file=/var/lib/minikube/certs/sa.pub --service-cluster-ip-range=10.96.0.0/12 --tls-cert-file=/var/lib/minikube/certs/apiserver.crt --tls-private-key-file=/var/lib/minikube/certs/apiserver.key
           │   │ ├─crio-conmon-da45f694e1afffe3d6cbd945e96f9798b34a728243ecd5232fc329f64446bd2d
           │   │ │ └─3093 /usr/libexec/crio/conmon --syslog -c da45f694e1afffe3d6cbd945e96f9798b34a728243ecd5232fc329f64446bd2d -n k8s_POD_kube-apiserver-minikube_kube-system_288cbfbd5565f92cc63b18bcafb084d5_0 -u da45f694e1afffe3d6cbd945e96f9798b34a728243ecd5232fc329f64446bd2d -r /usr/bin/runc -b /var/run/containers/storage/overlay-containers/da45f694e1afffe3d6cbd945e96f9798b34a728243ecd5232fc329f64446bd2d/userdata -p /var/run/containers/storage/overlay-containers/da45fb084d5/da45f694e1afffe3d6cbd945e96f9798b34a728243ecd5232fc329f64446bd2d.log --exit-dir /var/run/crio/exits --socket-dir-path /var/run/crio --log-level debug --runtime-arg --root=/run/runc --no-pivot
           │   │ └─crio-da45f694e1afffe3d6cbd945e96f9798b34a728243ecd5232fc329f64446bd2d
           │   │   └─3124 /pause
           │   ├─pod74dea8da17aa6241e5e4f7b2ba4e1d8e
           │   │ ├─crio-993cc803be23925d62509f515bf2b3c82defd9c6aaf02ffdfb66f14764396799
           │   │ │ └─3302 kube-scheduler --authentication-kubeconfig=/etc/kubernetes/scheduler.conf --authorization-kubeconfig=/etc/kubernetes/scheduler.conf --bind-address=127.0.0.1 --kubeconfig=/etc/kubernetes/scheduler.conf --leader-elect=true
           │   │ ├─crio-conmon-42954baf61955ccb20af54ca4ceb0d29e04c1661a7252c1555aeaa64b75da42f
           │   │ │ └─3110 /usr/libexec/crio/conmon --syslog -c 42954baf61955ccb20af54ca4ceb0d29e04c1661a7252c1555aeaa64b75da42f -n k8s_POD_kube-scheduler-minikube_kube-system_74dea8da17aa6241e5e4f7b2ba4e1d8e_0 -u 42954baf61955ccb20af54ca4ceb0d29e04c1661a7252c1555aeaa64b75da42f -r /usr/bin/runc -b /var/run/containers/storage/overlay-containers/42954baf61955ccb20af54ca4ceb0d29e04c1661a7252c1555aeaa64b75da42f/userdata -p /var/run/containers/storage/overlay-containers/429544e1d8e/42954baf61955ccb20af54ca4ceb0d29e04c1661a7252c1555aeaa64b75da42f.log --exit-dir /var/run/crio/exits --socket-dir-path /var/run/crio --log-level debug --runtime-arg --root=/run/runc --no-pivot
           │   │ ├─crio-42954baf61955ccb20af54ca4ceb0d29e04c1661a7252c1555aeaa64b75da42f
           │   │ │ └─3147 /pause
           │   │ └─crio-conmon-993cc803be23925d62509f515bf2b3c82defd9c6aaf02ffdfb66f14764396799
           │   │   └─3278 /usr/libexec/crio/conmon --syslog -c 993cc803be23925d62509f515bf2b3c82defd9c6aaf02ffdfb66f14764396799 -n k8s_kube-scheduler_kube-scheduler-minikube_kube-system_74dea8da17aa6241e5e4f7b2ba4e1d8e_0 -u 993cc803be23925d62509f515bf2b3c82defd9c6aaf02ffdfb66f14764396799 -r /usr/bin/runc -b /var/run/containers/storage/overlay-containers/993cc803be23925d62509f515bf2b3c82defd9c6aaf02ffdfb66f14764396799/userdata -p /var/run/containers/storage/overlay-containers/993cc803be23925d62509f515bf2b3c82defd9c6aaf02ffdfb66f14764396799/userdata/pidfile -l /var/log/pods/kube-system_kube-scheduler-minikube_74dea8da17aa6241e5e4f7b2ba4e1d8e/kube-scheduler/0.log --exit-dir /var/run/crio/exits --socket-dir-path /var/run/crio --log-level debug --runtime-arg --root=/run/runc --no-pivot
           │   ├─poda551ce90-e9ae-412e-b777-2a622a41b3d1
           │   │ ├─crio-42bf005600a5ce7b8b99d371a69fbd64bba8fcd112972ef43a7ee5d71ca18897
           │   │ │ └─3739 /pause
06708faeafa70077d626278-conmon-0c70b067f98ddadd639a90b776e6947a7d083b66a--More-- 
           │   │ │ └─3915 /usr/libexec/crio/conmon --syslog -c 0c70b067f98ddadd639a90b776e6947a7d083b66a06708faeafa70077d626278 -n k8s_coredns_coredns-5644d7b6d9-lgt6b_kube-system_a551ce90-e9ae-412e-b777-2a622a41b3d1_0 -u 0c70b067f98ddadd639a90b776e6947a7d083b66a06708faeafa70077d626278 -r /usr/bin/runc -b /var/run/containers/storage/overlay-containers/0c70b067f98ddadd639a90b776e6947a7d083b66a06708faeafa70077d626278/userdata -p /var/run/containers/storage/overlay-containers/0c70b067f98ddadd639a90b776e6947a7d083b66a06708faeafa70077d626278/userdata/pidfile -l /var/log/pods/kube-system_coredns-5644d7b6d9-lgt6b_a551ce90-e9ae-412e-b777-2a622a41b3d1/coredns/0.log --exit-dir /var/run/crio/exits --socket-dir-path /var/run/crio --log-level debug --runtime-arg --root=/run/runc --no-pivot
           │   │ ├─crio-0c70b067f98ddadd639a90b776e6947a7d083b66a06708faeafa70077d626278
           │   │ │ └─3926 /coredns -conf /etc/coredns/Corefile
           │   │ └─crio-conmon-42bf005600a5ce7b8b99d371a69fbd64bba8fcd112972ef43a7ee5d71ca18897
           │   │   └─3714 /usr/libexec/crio/conmon --syslog -c 42bf005600a5ce7b8b99d371a69fbd64bba8fcd112972ef43a7ee5d71ca18897 -n k8s_POD_coredns-5644d7b6d9-lgt6b_kube-system_a551ce90-e9ae-412e-b777-2a622a41b3d1_0 -u 42bf005600a5ce7b8b99d371a69fbd64bba8fcd112972ef43a7ee5d71ca18897 -r /usr/bin/runc -b /var/run/containers/storage/overlay-containers/42bf005600a5ce7b8b99d371a69fbd64bba8fcd112972ef43a7ee5d71ca18897/userdata -p /var/run/containers/storage/overlay-containers/ile -l /var/log/pods/kube-system_coredns-5644d7b6d9-lgt6b_a551ce90-e9ae-412e-b777-2a622a41b3d1/42bf005600a5ce7b8b99d371a69fbd64bba8fcd112972ef43a7ee5d71ca18897.log --exit-dir /var/run/crio/exits --socket-dir-path /var/run/crio --log-level debug --runtime-arg --root=/run/runc --no-pivot
           │   ├─podc3e29047da86ce6690916750ab69c40b
           │   │ ├─crio-conmon-0a85d1a6edd833057f8537f9195fb1aebb0273febb33790a996b58b64cb9bf50
           │   │ │ └─3061 /usr/libexec/crio/conmon --syslog -c 0a85d1a6edd833057f8537f9195fb1aebb0273febb33790a996b58b64cb9bf50 -n k8s_POD_kube-addon-manager-minikube_kube-system_c3e29047da86ce6690916750ab69c40b_0 -u 0a85d1a6edd833057f8537f9195fb1aebb0273febb33790a996b58b64cb9bf50 -r /usr/bin/runc -b /var/run/containers/storage/overlay-containers/0a85d1a6edd833057f8537f9195fb1aebb0273febb33790a996b58b64cb9bf50/userdata -p /var/run/containers/storage/overlay-containers/0a85d1a6edd833057f8537f9195fb1aebb0273febb33790a996b58b64cb9bf50/userdata/pidfile -l /var/log/pods/kube-system_kube-addon-manager-minikube_c3e29047da86ce6690916750ab69c40b/0a85d1a6edd833057f8537f9195fb1aebb0273febb33790a996b58b64cb9bf50.log --exit-dir /var/run/crio/exits --socket-dir-path /var/run/crio --log-level debug --runtime-arg --root=/run/runc --no-pivot
           │   │ ├─crio-conmon-5935820f335661c4ce86e9ef5f97a910b0958f942b2673cf224ee7b80b0c15f2
           │   │ │ └─3416 /usr/libexec/crio/conmon --syslog -c 5935820f335661c4ce86e9ef5f97a910b0958f942b2673cf224ee7b80b0c15f2 -n k8s_kube-addon-manager_kube-addon-manager-minikube_kube-system_c3e29047da86ce6690916750ab69c40b_0 ---More-u 5935820f335661c4ce86e9ef5f97a910b0958f942b2673cf224ee7b80b0c15f2 -r /usr/bin/runc -b /var/run/containers/storage/overlay-containers/5935820f335661c4ce86e9ef5f97a910b0958f942b2673cf224ee7b80b0c15f2/userdata -p /var/run/containers/storage/overlay-containers/5935820f335661c4ce86e9ef5f97a910b0958f942b2673cf224ee7b80b0c15f2/userdata/pidfile -l /var/log/pods/kube-system_kube-addon-manager-minikube_c3e29047da86ce6690916750ab69c40b/kube-addon-manager/0.log --exit-dir /var/run/crio/exits --socket-dir-path /var/run/crio --log-level debug --runtime-arg --root=/run/runc --no-pivot
           │   │ ├─crio-0a85d1a6edd833057f8537f9195fb1aebb0273febb33790a996b58b64cb9bf50
           │   │ │ └─3072 /pause
           │   │ └─crio-5935820f335661c4ce86e9ef5f97a910b0958f942b2673cf224ee7b80b0c15f2
           │   │   ├─3427 bash /opt/kube-addons.sh
           │   │   └─5230 sleep 4
           │   └─poddd190f17-7822-4345-9141-22a797802e0f
           │     ├─crio-conmon-1759f8374de60691c19247b1f93dab3f4c42b25e744c8350001de00524a7e059
           │     │ └─3724 /usr/libexec/crio/conmon --syslog -c 1759f8374de60691c19247b1f93dab3f4c42b25e744c8350001de00524a7e059 -n k8s_POD_coredns-5644d7b6d9-26x6p_kube-system_dd190f17-7822-4345-9141-22a797802e0f_0 -u 1759f8374de60691c194c8350001de00524a7e059/userdata -p /var/run/containers/storage/overlay-containers/1759f8374de60691c19247b1f93dab3f4c42b25e744c8350001de00524a7e059/userdata/pidfile -l /var/log/pods/kube-system_coredns-5644d7b6d9-26x6p_dd190f17-7822-4345-9141-22a797802e0f/1759f8374de60691c19247b1f93dab3f4c42b25e744c8350001de00524a7e059.log --exit-dir /var/run/crio/exits --socket-dir-path /var/run/crio --log-level debug --runtime-arg --root=/run/runc --no-pivot
           │     ├─crio-conmon-892f3d3545ee21ad95a171ae77636fe40c3983f0566d1439939a04430d64adb0
           │     │ └─3969 /usr/libexec/crio/conmon --syslog -c 892f3d3545ee21ad95a171ae77636fe40c3983f0566d1439939a04430d64adb0 -n k8s_coredns_coredns-5644d7b6d9-26x6p_kube-system_dd190f17-7822-4345-9141-22a797802e0f_0 -u 892f3d3545ee21ad95a171ae77636fe40c3983f0566d1439939a04430d64adb0 -r /usr/bin/runc -b /var/run/containers/storage/overlay-containers/892f3d3545ee21ad95a171ae77636fe40c3983f0566d1439939a04430d64adb0/userdata -p /var/run/containers/storage/overlay-containers/892f3d3545ee21ad95a171ae77636fe40c3983f0566d1439939a04430d64adb0/userdata/pidfile -l /var/log/pods/kube-system_coredns-5644d7b6d9-26x6p_dd190f17-7822-4345-9141-22a797802e0f/coredns/0.log --exit-dir /var/run/crio/exits --socket-dir-path /var/run/crio --log-level debug --runtime-arg --root=/run/runc --no-pivot
           │     ├─crio-1759f8374de60691c19247b1f93dab3f4c42b25e744c8350001de00524a7e059
           │     │ └─3760 /pause
           │     └─crio-892f3d3545ee21ad95a171ae77636fe40c3983f0566d1439939a04430d64adb0
           │       └─3980 /coredns -conf /etc/coredns/Corefile
           └─system.slice
             ├─nfs-mountd.service
             │ └─1660 /usr/sbin/rpc.mountd
             ├─systemd-timesyncd.service
             │ └─1636 /usr/lib/systemd/systemd-timesyncd
             ├─crio.service
             │ └─2049 /usr/bin/crio --log-level=debug --insecure-registry 10.96.0.0/12
             ├─dbus.service
             │ └─1685 /usr/bin/dbus-daemon --system --address=systemd: --nofork --nopidfile --systemd-activation --syslog-only
             ├─sshd.service
             │ └─1770 /usr/sbin/sshd -D -e
             ├─kubelet.service
             │ └─3011 /var/lib/minikube/binaries/v1.16.2/kubelet --authorization-mode=Webhook --bootstrap-kubeconfig=/etc/kubernetes/bootstrap-kubelet.conf --cgroup-driver=cgroupfs --client-ca-file=/var/lib/minikube/certs/ca.crt --cluster-dns=10.96.0.10 --cluster-domain=cluster.local --container-runtime=remote --container-runtime-endpoint=/var/run/crio/crio.sock --fail-swap-on=false --hostname-override=minikube --image-service-endpoint=/var/run/crio/crio.sock --kubeconfig=/etc/kubernetes/kubelet.conf --network-plugin=cni --node-ip=192.168.39.214 --pod-manifest-path=/etc/kubernetes/manifests --runtime-request-timeout=15m
             ├─system-serial\x2dgetty.slice
             │ └─serial-getty@ttyS0.service
             │   ├─1678 -sh
             │   ├─5263 systemctl status
             │   └─5264 /bin/more
             ├─system-getty.slice
             │ └─getty@tty1.service
             │   └─1679 /sbin/getty -L tty1 115200 vt100
             ├─rpcbind.service
             │ └─1677 /usr/bin/rpcbind
             ├─systemd-logind.service
             │ └─1688 /usr/lib/systemd/systemd-logind
             ├─systemd-resolved.service
             │ └─1656 /usr/lib/systemd/systemd-resolved
             ├─systemd-udevd.service
             │ └─1640 /usr/lib/systemd/systemd-udevd
             ├─systemd-journald.service
             │ └─1093 /usr/lib/systemd/systemd-journald
             └─systemd-networkd.service
               └─1649 /usr/lib/systemd/systemd-networkd

* Viewing CRIO images using the following command
        
        crictl images
        
        
        IMAGE                                     TAG                 IMAGE ID            SIZE
        gcr.io/k8s-minikube/storage-provisioner   v1.8.1              4689081edb103       80.8MB
        k8s.gcr.io/coredns                        1.6.2               bf261d1579144       44.2MB
        k8s.gcr.io/etcd                           3.3.15-0            b2756210eeabf       248MB
        k8s.gcr.io/k8s-dns-dnsmasq-nanny-amd64    1.14.13             6dc8ef8287d38       41.6MB
        k8s.gcr.io/k8s-dns-kube-dns-amd64         1.14.13             55a3c5209c5ea       51.4MB
        k8s.gcr.io/k8s-dns-sidecar-amd64          1.14.13             4b2e93f0133d3       43.1MB
        k8s.gcr.io/kube-addon-manager             v9.0                119701e77cbc4       84.7MB
        k8s.gcr.io/kube-addon-manager             v9.0.2              bd12a212f9dcb       84.7MB
        k8s.gcr.io/kube-apiserver                 v1.16.2             c2c9a0406787c       219MB
        k8s.gcr.io/kube-controller-manager        v1.16.2             6e4bffa46d70b       165MB
        k8s.gcr.io/kube-proxy                     v1.16.2             8454cbe08dc9f       87.9MB
        k8s.gcr.io/kube-scheduler                 v1.16.2             ebac1ae204a2c       88.8MB
        k8s.gcr.io/kubernetes-dashboard-amd64     v1.10.1             f9aed6605b814       122MB
        k8s.gcr.io/pause                          3.1                 da86e6ba6ca19       747kB


* Viewing running container

        crictl  ps
        
        
CONTAINER           IMAGE                                                                                                   CREATED             STATE               NAME                      ATTEMPT             POD ID
892f3d3545ee2       bf261d157914477ee1a5969d28ec687f3fbfc9fb5a664b22df78e57023b0e03b                                        7 minutes ago       Running             coredns                   0                   1759f8374de60
0c70b067f98dd       bf261d157914477ee1a5969d28ec687f3fbfc9fb5a664b22df78e57023b0e03b                                        7 minutes ago       Running             coredns                   0                   42bf005600a5c
21627aa0b5472       4689081edb103a9e8174bf23a255bfbe0b2d9ed82edc907abab6989d1c60f02c                                        7 minutes ago       Running             storage-provisioner       0                   ba42f9d76d115
f25ed7b9fb318       8454cbe08dc9ff820283939d1e509af7746256a3fc0511999c7e29ed8f29f852                                        7 minutes ago       Running             kube-proxy                0                   cb4126fb3d539
5935820f33566       k8s.gcr.io/kube-addon-manager@sha256:3e315022a842d782a28e729720f21091dde21f1efea28868d65ec595ad871616   7 minutes ago       Running             kube-addon-manager        0                   0a85d1a6edd83
42866e6955f0f       b2756210eeabf84f3221da9959e9483f3919dc2aaab4cd45e7cd072fcbde27ed                                        8 minutes ago       Running             etcd                      0                   fa20492d16fce
993cc803be239       ebac1ae204a2c8d36411792d8bfea34bc82bd733b499ff81ca10e0f6c082d44c                                        8 minutes ago       Running             kube-scheduler            0                   42954baf61955
90f7b48799d7f       6e4bffa46d70bb06f05f9e32d9f73511fe59c0dcd85aa25a56e651c1250b4777                                        8 minutes ago       Running             kube-controller-manager   0                   027e7f3d79249
8fedc929a6c2f       c2c9a0406787cbb6b206d082f1085215e9b0c98483b6e3dba3d0df6d4653e7b0                                        8 minutes ago       Running             kube-apiserver            0                   da45f694e1aff

* Few examples available from client-go project that helped to play around with minikube
    * client-go/examples/
    
* json structure for Kubernetes can be found inside k8s.io/kubernetes/staging/src/k8s.io/api/core/v1/types.go

* testing
    * the test cases use a lot of 'fake' classes that are made available inside k8s.io/client-go/kubernetes/typed/core/v1/fake folder
    
    
* API Server
    * For example to access a pod's log use the following command
        
        curl -k --cert ~/.minikube/apiserver.crt --key ~/.minikube/apiserver.key  -v -XGET https://192.168.99.223:8443/api/v1/namespaces/default/pods/demo-788cf8d6f5-6d6qz/log

    
* pkg/minikube/service/service.go --> contains code to connect to kubernetes 

    type K8sClient interface {
        GetCoreClient() (typed_core.CoreV1Interface, error)
        GetClientset(timeout time.Duration) (*kubernetes.Clientset, error)
    }
