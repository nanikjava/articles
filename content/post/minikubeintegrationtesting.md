---
date: "2019-12-01"
title: "Minikube Integration Testing (Work in progress)"
author : "Nanik Tolaram (nanikjava@gmail.com)" 

---
This article will outline few things that are helpful when running or modifying minikube testing code.

* Make sure **kubectl** is in the PATH
* Following command is to disable parallel testing and enable all log output

    > -parallel 1  -test.v
    
    the **test.v** parameter instruct the testing framework to output non-failure messages. The output will not be realtime as it will only be outputted when the test complete.
  
* To run the test use the following command
    
    > PATH=/home/nanik/Golang/go/bin:/home/nanik/Downloads:/home/nanik/bin:/usr/local/sbin:/usr/local/bin:/usr/sbin:/usr/bin:/sbin:/bin:/usr/games:/usr/local/games:/snap/bin:/snap/bin:/var/lib/snapd/snap/bin:/home/nanik/Downloads/ go test   -test.v -tags integration

    The **/home/nanik/Downloads** is added to the PATH because that's the location for kubectl and the test cases uses kubectl. 

    DO NOT USE _**-parrallel 1**_ when running ALL the test cases as for some reason it throws an error.


* Running **make integration** command will yield output similar to the following 

{{< highlight text >}}
GOOS="linux" GOARCH="amd64" go build -tags "container_image_ostree_stub containers_image_openpgp go_getter_nos3 go_getter_nogcs" -ldflags="-X k8s.io/minikube/pkg/version.version=v1.5.2 -X k8s.io/minikube/pkg/version.isoVersion=v1.5.1 -X k8s.io/minikube/pkg/version.isoPath=minikube/iso -X k8s.io/minikube/pkg/version.gitCommitID="3322c50ceb4abf795ebb8505f99bbb269a144e21-dirty"" -a -o out/minikube-linux-amd64 k8s.io/minikube/cmd/minikube

cp out/minikube-linux-amd64 out/minikube
go test -v -test.timeout=60m ./test/integration --tags="integration container_image_ostree_stub containers_image_openpgp go_getter_nos3 go_getter_nogcs" 
=== RUN   TestDownloadOnly
=== RUN   TestDownloadOnly/group
=== RUN   TestDownloadOnly/group/v1.11.10
=== RUN   TestDownloadOnly/group/v1.16.2
=== RUN   TestDownloadOnly/group/v1.16.2#01
=== RUN   TestDownloadOnly/group/ExpectedDefaultDriver
=== RUN   TestDownloadOnly/group/DeleteAll
=== RUN   TestDownloadOnly/group/DeleteAlwaysSucceeds
--- PASS: TestDownloadOnly (33.63s)
    --- PASS: TestDownloadOnly/group (33.57s)
        --- PASS: TestDownloadOnly/group/v1.11.10 (19.74s)
            a_serial_test.go:61: (dbg) Run:  out/minikube start --download-only -p download-20191104T082052.238721463-18873 --force --alsologtostderr --kubernetes-version=v1.11.10
            a_serial_test.go:61: (dbg) Done: out/minikube start --download-only -p download-20191104T082052.238721463-18873 --force --alsologtostderr --kubernetes-version=v1.11.10: (19.741382231s)
        --- PASS: TestDownloadOnly/group/v1.16.2 (13.46s)
            a_serial_test.go:63: (dbg) Run:  out/minikube start --download-only -p download-20191104T082052.238721463-18873 --force --alsologtostderr --kubernetes-version=v1.16.2
            a_serial_test.go:63: (dbg) Done: out/minikube start --download-only -p download-20191104T082052.238721463-18873 --force --alsologtostderr --kubernetes-version=v1.16.2: (13.460844164s)
        --- PASS: TestDownloadOnly/group/v1.16.2#01 (0.27s)
            a_serial_test.go:63: (dbg) Run:  out/minikube start --download-only -p download-20191104T082052.238721463-18873 --force --alsologtostderr --kubernetes-version=v1.16.2
        --- SKIP: TestDownloadOnly/group/ExpectedDefaultDriver (0.00s)
            a_serial_test.go:94: --expected-default-driver is unset, skipping test
        --- PASS: TestDownloadOnly/group/DeleteAll (0.06s)
            a_serial_test.go:124: (dbg) Run:  out/minikube delete --all
        --- PASS: TestDownloadOnly/group/DeleteAlwaysSucceeds (0.03s)
            a_serial_test.go:134: (dbg) Run:  out/minikube delete -p download-20191104T082052.238721463-18873
    helpers.go:167: (dbg) Run:  out/minikube delete -p download-20191104T082052.238721463-18873
=== RUN   TestAddons
=== PAUSE TestAddons
=== RUN   TestDockerFlags
=== PAUSE TestDockerFlags
=== RUN   TestKVMDriverInstallOrUpdate
=== PAUSE TestKVMDriverInstallOrUpdate
=== RUN   TestHyperKitDriverInstallOrUpdate
--- SKIP: TestHyperKitDriverInstallOrUpdate (0.00s)
    driver_install_or_update_test.go:101: Skip if not darwin.
=== RUN   TestFunctional
=== RUN   TestFunctional/serial
=== RUN   TestFunctional/serial/StartWithProxy
=== RUN   TestFunctional/serial/KubeContext
=== RUN   TestFunctional/serial/KubectlGetPods
=== RUN   TestFunctional/serial/CacheCmd
=== PAUSE TestFunctional
=== RUN   TestGvisorAddon
--- SKIP: TestGvisorAddon (0.00s)
    gvisor_addon_test.go:34: skipping test because --gvisor=false
=== RUN   TestChangeNoneUser
--- SKIP: TestChangeNoneUser (0.00s)
    none_test.go:39: Only test none driver.
=== RUN   TestStartStop
=== PAUSE TestStartStop
=== RUN   TestVersionUpgrade
=== PAUSE TestVersionUpgrade
=== CONT  TestAddons
=== CONT  TestStartStop
=== CONT  TestFunctional
=== CONT  TestVersionUpgrade
=== CONT  TestDockerFlags
=== RUN   TestFunctional/parallel
=== CONT  TestKVMDriverInstallOrUpdate
=== RUN   TestFunctional/parallel/AddonManager
=== RUN   TestStartStop/group
=== PAUSE TestFunctional/parallel/AddonManager
=== RUN   TestFunctional/parallel/ComponentHealth
=== PAUSE TestFunctional/parallel/ComponentHealth
=== RUN   TestStartStop/group/docker
=== RUN   TestFunctional/parallel/ConfigCmd
=== PAUSE TestFunctional/parallel/ConfigCmd
=== RUN   TestFunctional/parallel/DashboardCmd
=== PAUSE TestStartStop/group/docker
=== RUN   TestStartStop/group/cni
=== PAUSE TestFunctional/parallel/DashboardCmd
=== RUN   TestFunctional/parallel/DNS
=== PAUSE TestStartStop/group/cni
=== PAUSE TestFunctional/parallel/DNS
=== RUN   TestStartStop/group/containerd
=== RUN   TestFunctional/parallel/StatusCmd
=== PAUSE TestFunctional/parallel/StatusCmd
=== PAUSE TestStartStop/group/containerd
=== RUN   TestFunctional/parallel/LogsCmd
=== RUN   TestStartStop/group/crio
=== PAUSE TestFunctional/parallel/LogsCmd
=== RUN   TestFunctional/parallel/MountCmd
=== PAUSE TestStartStop/group/crio
=== CONT  TestStartStop/group/docker
=== PAUSE TestFunctional/parallel/MountCmd
=== CONT  TestStartStop/group/containerd
=== RUN   TestFunctional/parallel/ProfileCmd
=== CONT  TestStartStop/group/crio
=== CONT  TestStartStop/group/cni
=== PAUSE TestFunctional/parallel/ProfileCmd
=== RUN   TestFunctional/parallel/ServiceCmd
=== PAUSE TestFunctional/parallel/ServiceCmd
=== RUN   TestFunctional/parallel/AddonsCmd
=== PAUSE TestFunctional/parallel/AddonsCmd
=== RUN   TestFunctional/parallel/PersistentVolumeClaim
=== PAUSE TestFunctional/parallel/PersistentVolumeClaim
=== RUN   TestFunctional/parallel/TunnelCmd
=== PAUSE TestFunctional/parallel/TunnelCmd
=== RUN   TestFunctional/parallel/SSHCmd
=== PAUSE TestFunctional/parallel/SSHCmd
=== RUN   TestFunctional/parallel/MySQL
=== PAUSE TestFunctional/parallel/MySQL
=== CONT  TestFunctional/parallel/AddonManager
=== CONT  TestFunctional/parallel/MySQL
=== CONT  TestFunctional/parallel/MountCmd
=== CONT  TestFunctional/parallel/DNS
    > docker-machine-driver-kvm2.sha256: 65 B / 65 B [-------] 100.00% ? p/s 0s
=== CONT  TestFunctional/parallel/ConfigCmd
=== CONT  TestFunctional/parallel/DashboardCmd
=== CONT  TestFunctional/parallel/LogsCmd
=== CONT  TestFunctional/parallel/ComponentHealth
=== CONT  TestFunctional/parallel/StatusCmd
=== CONT  TestFunctional/parallel/PersistentVolumeClaim
    > docker-machine-driver-kvm2: 0 B / 48.57 MiB [_____________] 0.00% ? p/s ?    > docker-machine-driver-kvm2: 33.55 KiB / 48.57 MiB [>______] 0.07% ? p/s ?    > docker-machine-driver-kvm2: 84.55 KiB / 48.57 MiB [>______] 0.17% ? p/s ?    > docker-machine-driver-kvm2: 169.55 KiB / 48.57 MiB  0.34% 282.61 KiB p/s     > docker-machine-driver-kvm2: 169.55 KiB / 48.57 MiB  0.34% 282.61 KiB p/s     > docker-machine-driver-kvm2: 338.52 KiB / 48.57 MiB  0.68% 282.61 KiB p/s     > docker-machine-driver-kvm2: 662.55 KiB / 48.57 MiB  1.33% 317.38 KiB p/s === CONT  TestFunctional/parallel/SSHCmd
    > docker-machine-driver-kvm2: 662.55 KiB / 48.57 MiB  1.33% 317.38 KiB p/s === CONT  TestFunctional/parallel/TunnelCmd
=== CONT  TestFunctional/parallel/ServiceCmd
    > docker-machine-driver-kvm2: 1.31 MiB / 48.57 MiB  2.70% 317.38 KiB p/s ET    > docker-machine-driver-kvm2: 2.13 MiB / 48.57 MiB  4.38% 459.99 KiB p/s ET    > docker-machine-driver-kvm2: 2.30 MiB / 48.57 MiB  4.74% 459.99 KiB p/s ET    > docker-machine-driver-kvm2: 3.35 MiB / 48.57 MiB  6.89% 459.99 KiB p/s ET    > docker-machine-driver-kvm2: 4.44 MiB / 48.57 MiB  9.14% 684.97 KiB p/s ET    >      > .....
    > .....
=== CONT  TestFunctional/parallel/AddonsCmd
=== CONT  TestFunctional/parallel/ProfileCmd
    > docker-machine-driver-kvm2: 21.21 MiB / 48.57 MiB  43.66% 1.86 MiB p/s ET    > docker-machine-driver-kvm2: 21.21 MiB / 48.57 MiB  43.66% 2.08 MiB p/s ET    > docker-machine-driver-kvm2: 22.78 MiB / 48.57 MiB  46.91% 2.08 MiB p/s ET    > docker-machine-driver-kvm2: 23.35 MiB / 48.57 MiB  48.06% 2.08 MiB p/s ET    > docker-machine-driver-kvm2: 24.91 MiB / 48.57 MiB  51.28% 2.34 MiB p/s ET   
     > .....
     > .....
     > .....    
2019/11/04 08:25:30 [DEBUG] GET http://192.168.39.24:31849
2019/11/04 08:25:38 [DEBUG] GET http://127.0.0.1:38193/api/v1/namespaces/kubernetes-dashboard/services/http:kubernetes-dashboard:/proxy/
--- PASS: TestFunctional (254.26s)
    --- PASS: TestFunctional/serial (206.43s)
        --- PASS: TestFunctional/serial/StartWithProxy (177.80s)
            functional_test.go:120: (dbg) Run:  out/minikube start -p functional-20191104T082125.872658733-18873 --wait=false 
            functional_test.go:120: (dbg) Done: out/minikube start -p functional-20191104T082125.872658733-18873 --wait=false : (2m57.799561087s)
        --- PASS: TestFunctional/serial/KubeContext (0.05s)
            functional_test.go:138: (dbg) Run:  kubectl config current-context
        --- PASS: TestFunctional/serial/KubectlGetPods (0.44s)
            functional_test.go:149: (dbg) Run:  kubectl get pod -A
        --- PASS: TestFunctional/serial/CacheCmd (28.14s)
            functional_test.go:302: (dbg) Run:  out/minikube -p functional-20191104T082125.872658733-18873 cache add busybox
            functional_test.go:302: (dbg) Done: out/minikube -p functional-20191104T082125.872658733-18873 cache add busybox: (5.119077944s)
            functional_test.go:302: (dbg) Run:  out/minikube -p functional-20191104T082125.872658733-18873 cache add busybox:1.28.4-glibc
            functional_test.go:302: (dbg) Done: out/minikube -p functional-20191104T082125.872658733-18873 cache add busybox:1.28.4-glibc: (5.002939096s)
            functional_test.go:302: (dbg) Run:  out/minikube -p functional-20191104T082125.872658733-18873 cache add mysql:5.6
            functional_test.go:302: (dbg) Done: out/minikube -p functional-20191104T082125.872658733-18873 cache add mysql:5.6: (18.018114782s)
    --- PASS: TestFunctional/parallel (0.00s)
        --- PASS: TestFunctional/parallel/MountCmd (4.77s)
            fn_mount_cmd.go:62: (dbg) daemon: [../../out/minikube mount -p functional-20191104T082125.872658733-18873 /tmp/mounttest209263981:/mount-9p --alsologtostderr -v=1]
            fn_mount_cmd.go:96: wrote "test-1572816292301508417" to /tmp/mounttest209263981/created-by-test
            fn_mount_cmd.go:96: wrote "test-1572816292301508417" to /tmp/mounttest209263981/created-by-test-removed-by-pod
            fn_mount_cmd.go:96: wrote "test-1572816292301508417" to /tmp/mounttest209263981/test-1572816292301508417
            fn_mount_cmd.go:104: (dbg) Run:  out/minikube -p functional-20191104T082125.872658733-18873 ssh "findmnt -T /mount-9p | grep 9p"
            fn_mount_cmd.go:104: (dbg) Non-zero exit: out/minikube -p functional-20191104T082125.872658733-18873 ssh "findmnt -T /mount-9p | grep 9p": exit status 1 (202.701275ms)
                
                ** stderr ** 
                ssh: Process exited with status 1
                
                ** /stderr **
            fn_mount_cmd.go:104: (dbg) Run:  out/minikube -p functional-20191104T082125.872658733-18873 ssh "findmnt -T /mount-9p | grep 9p"
            fn_mount_cmd.go:118: (dbg) Run:  out/minikube -p functional-20191104T082125.872658733-18873 ssh -- ls -la /mount-9p
            fn_mount_cmd.go:122: guest mount directory contents
                total 2
                -rw-r--r-- 1 docker docker 24 Nov  3 21:24 created-by-test
                -rw-r--r-- 1 docker docker 24 Nov  3 21:24 created-by-test-removed-by-pod
                -rw-r--r-- 1 docker docker 24 Nov  3 21:24 test-1572816292301508417
            fn_mount_cmd.go:126: (dbg) Run:  out/minikube -p functional-20191104T082125.872658733-18873 ssh cat /mount-9p/test-1572816292301508417
            fn_mount_cmd.go:137: (dbg) Run:  kubectl --context functional-20191104T082125.872658733-18873 replace --force -f testdata/busybox-mount-test.yaml
            fn_mount_cmd.go:142: (dbg) waiting for pods with labels "integration-test=busybox-mount" in namespace "default" ...
            helpers.go:247: "busybox-mount" [cad31f9d-5fe3-423d-b72c-76365e32eaac] Pending
            helpers.go:247: "busybox-mount" [cad31f9d-5fe3-423d-b72c-76365e32eaac] Pending / Ready:ContainersNotReady (containers with unready status: [mount-munger]) / ContainersReady:ContainersNotReady (containers with unready status: [mount-munger])
            helpers.go:247: "busybox-mount" [cad31f9d-5fe3-423d-b72c-76365e32eaac] Succeeded: Initialized:PodCompleted / Ready:PodCompleted / ContainersReady:PodCompleted
            fn_mount_cmd.go:142: (dbg) pods integration-test=busybox-mount up and healthy within 2.006393812s
            fn_mount_cmd.go:158: (dbg) Run:  kubectl --context functional-20191104T082125.872658733-18873 logs busybox-mount
            fn_mount_cmd.go:170: (dbg) Run:  out/minikube -p functional-20191104T082125.872658733-18873 ssh stat /mount-9p/created-by-test
            fn_mount_cmd.go:170: (dbg) Run:  out/minikube -p functional-20191104T082125.872658733-18873 ssh stat /mount-9p/created-by-pod
            fn_mount_cmd.go:79: (dbg) Run:  out/minikube -p functional-20191104T082125.872658733-18873 ssh "sudo umount -f /mount-9p"
            fn_mount_cmd.go:83: (dbg) stopping [../../out/minikube mount -p functional-20191104T082125.872658733-18873 /tmp/mounttest209263981:/mount-9p --alsologtostderr -v=1] ...
        --- PASS: TestFunctional/parallel/ConfigCmd (0.15s)
            functional_test.go:326: (dbg) Run:  out/minikube -p functional-20191104T082125.872658733-18873 config unset cpus
            functional_test.go:326: (dbg) Run:  out/minikube -p functional-20191104T082125.872658733-18873 config get cpus
            functional_test.go:326: (dbg) Non-zero exit: out/minikube -p functional-20191104T082125.872658733-18873 config get cpus: exit status 64 (21.49557ms)
                
                ** stderr ** 
                Error: specified key could not be found in config
                
                ** /stderr **
            functional_test.go:326: (dbg) Run:  out/minikube -p functional-20191104T082125.872658733-18873 config set cpus 2
            functional_test.go:326: (dbg) Run:  out/minikube -p functional-20191104T082125.872658733-18873 config get cpus
            functional_test.go:326: (dbg) Run:  out/minikube -p functional-20191104T082125.872658733-18873 config unset cpus
            functional_test.go:326: (dbg) Run:  out/minikube -p functional-20191104T082125.872658733-18873 config get cpus
            functional_test.go:326: (dbg) Non-zero exit: out/minikube -p functional-20191104T082125.872658733-18873 config get cpus: exit status 64 (19.061597ms)
                
                ** stderr ** 
                Error: specified key could not be found in config
                
                ** /stderr **
        --- PASS: TestFunctional/parallel/AddonManager (5.06s)
            functional_test.go:162: (dbg) waiting for pods with labels "component=kube-addon-manager" in namespace "kube-system" ...
            helpers.go:247: "kube-addon-manager-minikube" [713e5c52-a374-40ed-9f57-f14c7d0de26a] Running
            functional_test.go:162: (dbg) pods component=kube-addon-manager up and healthy within 5.026986587s
        --- PASS: TestFunctional/parallel/LogsCmd (0.59s)
            functional_test.go:344: (dbg) Run:  out/minikube -p functional-20191104T082125.872658733-18873 logs
        --- PASS: TestFunctional/parallel/ComponentHealth (0.07s)
            functional_test.go:169: (dbg) Run:  kubectl --context functional-20191104T082125.872658733-18873 get cs -o=json
        --- PASS: TestFunctional/parallel/StatusCmd (0.57s)
            functional_test.go:194: (dbg) Run:  out/minikube -p functional-20191104T082125.872658733-18873 status
            functional_test.go:200: (dbg) Run:  out/minikube -p functional-20191104T082125.872658733-18873 status -f host:{{.Host}},kublet:{{.Kubelet}},apiserver:{{.APIServer}},kubeconfig:{{.Kubeconfig}}
            functional_test.go:210: (dbg) Run:  out/minikube -p functional-20191104T082125.872658733-18873 status -o json
        --- PASS: TestFunctional/parallel/DNS (8.60s)
            functional_test.go:270: (dbg) Run:  kubectl --context functional-20191104T082125.872658733-18873 replace --force -f testdata/busybox.yaml
            functional_test.go:275: (dbg) waiting for pods with labels "integration-test=busybox" in namespace "default" ...
            helpers.go:247: "busybox" [24600016-7278-4daa-8a5d-b54f465d0f19] Pending
            helpers.go:247: "busybox" [24600016-7278-4daa-8a5d-b54f465d0f19] Pending / Ready:ContainersNotReady (containers with unready status: [busybox]) / ContainersReady:ContainersNotReady (containers with unready status: [busybox])
            helpers.go:247: "busybox" [24600016-7278-4daa-8a5d-b54f465d0f19] Running
            functional_test.go:275: (dbg) pods integration-test=busybox up and healthy within 8.011521017s
            functional_test.go:281: (dbg) Run:  kubectl --context functional-20191104T082125.872658733-18873 exec busybox nslookup kubernetes.default
        --- PASS: TestFunctional/parallel/SSHCmd (0.15s)
            functional_test.go:537: (dbg) Run:  out/minikube -p functional-20191104T082125.872658733-18873 ssh "echo hello"
        --- SKIP: TestFunctional/parallel/TunnelCmd (0.01s)
            fn_tunnel_cmd.go:47: password required to execute 'route', skipping testTunnel: exit status 1
        --- PASS: TestFunctional/parallel/MySQL (12.36s)
            functional_test.go:548: (dbg) Run:  kubectl --context functional-20191104T082125.872658733-18873 replace --force -f testdata/mysql.yaml
            functional_test.go:555: (dbg) waiting for pods with labels "app=mysql" in namespace "default" ...
            helpers.go:247: "mysql-5787d7b5fc-kqdrj" [90a614ba-f2d8-4318-92ae-6edd7463f14c] Pending
            helpers.go:247: "mysql-5787d7b5fc-kqdrj" [90a614ba-f2d8-4318-92ae-6edd7463f14c] Pending / Ready:ContainersNotReady (containers with unready status: [mysql]) / ContainersReady:ContainersNotReady (containers with unready status: [mysql])
            helpers.go:247: "mysql-5787d7b5fc-kqdrj" [90a614ba-f2d8-4318-92ae-6edd7463f14c] Running
            functional_test.go:555: pod "app=mysql" failed to start: timed out waiting for the condition
            helpers.go:292: (dbg) Run:  kubectl --context functional-20191104T082125.872658733-18873 get po -A --show-labels
            helpers.go:298: (dbg) kubectl --context functional-20191104T082125.872658733-18873 get po -A --show-labels:
                NAMESPACE     NAME                               READY   STATUS      RESTARTS   AGE    LABELS
                default       busybox                            1/1     Running     0          5s     integration-test=busybox
                default       busybox-mount                      0/1     Completed   0          3s     integration-test=busybox-mount
                default       mysql-5787d7b5fc-kqdrj             1/1     Running     0          5s     app=mysql,pod-template-hash=5787d7b5fc
                kube-system   coredns-5644d7b6d9-p9dkz           1/1     Running     0          104s   k8s-app=kube-dns,pod-template-hash=5644d7b6d9
                kube-system   coredns-5644d7b6d9-wbttz           1/1     Running     0          104s   k8s-app=kube-dns,pod-template-hash=5644d7b6d9
                kube-system   etcd-minikube                      1/1     Running     0          41s    component=etcd,tier=control-plane
                kube-system   kube-addon-manager-minikube        1/1     Running     0          112s   component=kube-addon-manager,kubernetes.io/minikube-addons=addon-manager,version=v9.0.2
                kube-system   kube-apiserver-minikube            1/1     Running     0          34s    component=kube-apiserver,tier=control-plane
                kube-system   kube-controller-manager-minikube   1/1     Running     0          37s    component=kube-controller-manager,tier=control-plane
                kube-system   kube-proxy-wc5fw                   1/1     Running     0          104s   controller-revision-hash=56ffd4ff47,k8s-app=kube-proxy,pod-template-generation=1
                kube-system   kube-scheduler-minikube            1/1     Running     0          26s    component=kube-scheduler,tier=control-plane
                kube-system   storage-provisioner                1/1     Running     0          103s   addonmanager.kubernetes.io/mode=Reconcile,integration-test=storage-provisioner
            helpers.go:301: (dbg) Run:  kubectl --context functional-20191104T082125.872658733-18873 describe po mysql-5787d7b5fc-kqdrj -n default
            helpers.go:305: (dbg) kubectl --context functional-20191104T082125.872658733-18873 describe po mysql-5787d7b5fc-kqdrj -n default:
                Name:           mysql-5787d7b5fc-kqdrj
                Namespace:      default
                Priority:       0
                Node:           minikube/192.168.39.24
                Start Time:     Mon, 04 Nov 2019 08:24:52 +1100
                Labels:         app=mysql
                                pod-template-hash=5787d7b5fc
                Annotations:    <none>
                Status:         Running
                IP:             172.17.0.5
                Controlled By:  ReplicaSet/mysql-5787d7b5fc
                Containers:
                  mysql:
                    Container ID:   docker://38da7fcc7cc8841b3233ef5fc800c8375602336625daf18456b13eafb40ac7d9
                    Image:          mysql:5.6
                    Image ID:       docker://sha256:b3983abaa3feff23bf58b849c7c2c0f84862e3a749a3181a0bdd6fcfa023f318
                    Port:           3306/TCP
                    Host Port:      0/TCP
                    State:          Running
                      Started:      Mon, 04 Nov 2019 08:24:54 +1100
                    Ready:          True
                    Restart Count:  0
                    Environment:
                      MYSQL_ROOT_PASSWORD:  password
                    Mounts:
                      /var/run/secrets/kubernetes.io/serviceaccount from default-token-s7jh6 (ro)
                Conditions:
                  Type              Status
                  Initialized       True 
                  Ready             True 
                  ContainersReady   True 
                  PodScheduled      True 
                Volumes:
                  default-token-s7jh6:
                    Type:        Secret (a volume populated by a Secret)
                    SecretName:  default-token-s7jh6
                    Optional:    false
                QoS Class:       BestEffort
                Node-Selectors:  <none>
                Tolerations:     node.kubernetes.io/not-ready:NoExecute for 300s
                                 node.kubernetes.io/unreachable:NoExecute for 300s
                Events:
                  Type    Reason     Age        From               Message
                  ----    ------     ----       ----               -------
                  Normal  Scheduled  <unknown>  default-scheduler  Successfully assigned default/mysql-5787d7b5fc-kqdrj to minikube
                  Normal  Pulled     4s         kubelet, minikube  Container image "mysql:5.6" already present on machine
                  Normal  Created    4s         kubelet, minikube  Created container mysql
                  Normal  Started    3s         kubelet, minikube  Started container mysql
            helpers.go:308: (dbg) Run:  kubectl --context functional-20191104T082125.872658733-18873 logs mysql-5787d7b5fc-kqdrj -n default
            helpers.go:312: (dbg) kubectl --context functional-20191104T082125.872658733-18873 logs mysql-5787d7b5fc-kqdrj -n default:
                2019-11-03 21:24:54+00:00 [Note] [Entrypoint]: Entrypoint script for MySQL Server 5.6.46-1debian9 started.
                2019-11-03 21:24:54+00:00 [Note] [Entrypoint]: Switching to dedicated user 'mysql'
                2019-11-03 21:24:54+00:00 [Note] [Entrypoint]: Entrypoint script for MySQL Server 5.6.46-1debian9 started.
                2019-11-03 21:24:54+00:00 [Note] [Entrypoint]: Initializing database files
                2019-11-03 21:24:54 0 [Warning] TIMESTAMP with implicit DEFAULT value is deprecated. Please use --explicit_defaults_for_timestamp server option (see documentation for more details).
                2019-11-03 21:24:54 0 [Note] Ignoring --secure-file-priv value as server is running with --bootstrap.
                2019-11-03 21:24:54 0 [Note] /usr/sbin/mysqld (mysqld 5.6.46) starting as process 49 ...
                2019-11-03 21:24:54 49 [Note] InnoDB: Using atomics to ref count buffer pool pages
                2019-11-03 21:24:54 49 [Note] InnoDB: The InnoDB memory heap is disabled
                2019-11-03 21:24:54 49 [Note] InnoDB: Mutexes and rw_locks use GCC atomic builtins
                2019-11-03 21:24:54 49 [Note] InnoDB: Memory barrier is not used
                2019-11-03 21:24:54 49 [Note] InnoDB: Compressed tables use zlib 1.2.11
                2019-11-03 21:24:54 49 [Note] InnoDB: Using Linux native AIO
                2019-11-03 21:24:54 49 [Note] InnoDB: Using CPU crc32 instructions
                2019-11-03 21:24:54 49 [Note] InnoDB: Initializing buffer pool, size = 128.0M
                2019-11-03 21:24:54 49 [Note] InnoDB: Completed initialization of buffer pool
                2019-11-03 21:24:54 49 [Note] InnoDB: The first specified data file ./ibdata1 did not exist: a new database to be created!
                2019-11-03 21:24:54 49 [Note] InnoDB: Setting file ./ibdata1 size to 12 MB
                2019-11-03 21:24:54 49 [Note] InnoDB: Database physically writes the file full: wait...
                2019-11-03 21:24:54 49 [Note] InnoDB: Setting log file ./ib_logfile101 size to 48 MB
                2019-11-03 21:24:54 49 [Note] InnoDB: Setting log file ./ib_logfile1 size to 48 MB
                2019-11-03 21:24:55 49 [Note] InnoDB: Renaming log file ./ib_logfile101 to ./ib_logfile0
                2019-11-03 21:24:55 49 [Warning] InnoDB: New log files created, LSN=45781
                2019-11-03 21:24:55 49 [Note] InnoDB: Doublewrite buffer not found: creating new
                2019-11-03 21:24:55 49 [Note] InnoDB: Doublewrite buffer created
                2019-11-03 21:24:55 49 [Note] InnoDB: 128 rollback segment(s) are active.
                2019-11-03 21:24:55 49 [Warning] InnoDB: Creating foreign key constraint system tables.
                2019-11-03 21:24:55 49 [Note] InnoDB: Foreign key constraint system tables created
                2019-11-03 21:24:55 49 [Note] InnoDB: Creating tablespace and datafile system tables.
                2019-11-03 21:24:55 49 [Note] InnoDB: Tablespace and datafile system tables created.
                2019-11-03 21:24:55 49 [Note] InnoDB: Waiting for purge to start
                2019-11-03 21:24:55 49 [Note] InnoDB: 5.6.46 started; log sequence number 0
                2019-11-03 21:24:55 49 [Note] RSA private key file not found: /var/lib/mysql//private_key.pem. Some authentication plugins will not work.
                2019-11-03 21:24:55 49 [Note] RSA public key file not found: /var/lib/mysql//public_key.pem. Some authentication plugins will not work.
                2019-11-03 21:24:56 49 [Note] Binlog end
                2019-11-03 21:24:56 49 [Note] InnoDB: FTS optimize thread exiting.
                2019-11-03 21:24:56 49 [Note] InnoDB: Starting shutdown...
                2019-11-03 21:24:57 49 [Note] InnoDB: Shutdown completed; log sequence number 1625977
                
                
                2019-11-03 21:24:57 0 [Warning] TIMESTAMP with implicit DEFAULT value is deprecated. Please use --explicit_defaults_for_timestamp server option (see documentation for more details).
                2019-11-03 21:24:57 0 [Note] Ignoring --secure-file-priv value as server is running with --bootstrap.
                2019-11-03 21:24:57 0 [Note] /usr/sbin/mysqld (mysqld 5.6.46) starting as process 72 ...
                2019-11-03 21:24:57 72 [Note] InnoDB: Using atomics to ref count buffer pool pages
                2019-11-03 21:24:57 72 [Note] InnoDB: The InnoDB memory heap is disabled
                2019-11-03 21:24:57 72 [Note] InnoDB: Mutexes and rw_locks use GCC atomic builtins
                2019-11-03 21:24:57 72 [Note] InnoDB: Memory barrier is not used
                2019-11-03 21:24:57 72 [Note] InnoDB: Compressed tables use zlib 1.2.11
                2019-11-03 21:24:57 72 [Note] InnoDB: Using Linux native AIO
                2019-11-03 21:24:57 72 [Note] InnoDB: Using CPU crc32 instructions
                2019-11-03 21:24:57 72 [Note] InnoDB: Initializing buffer pool, size = 128.0M
                2019-11-03 21:24:57 72 [Note] InnoDB: Completed initialization of buffer pool
                2019-11-03 21:24:57 72 [Note] InnoDB: Highest supported file format is Barracuda.
                2019-11-03 21:24:57 72 [Note] InnoDB: 128 rollback segment(s) are active.
                2019-11-03 21:24:57 72 [Note] InnoDB: Waiting for purge to start
                2019-11-03 21:24:57 72 [Note] InnoDB: 5.6.46 started; log sequence number 1625977
                2019-11-03 21:24:57 72 [Note] RSA private key file not found: /var/lib/mysql//private_key.pem. Some authentication plugins will not work.
                2019-11-03 21:24:57 72 [Note] RSA public key file not found: /var/lib/mysql//public_key.pem. Some authentication plugins will not work.
                2019-11-03 21:24:57 72 [Note] Binlog end
                2019-11-03 21:24:57 72 [Note] InnoDB: FTS optimize thread exiting.
                2019-11-03 21:24:57 72 [Note] InnoDB: Starting shutdown...
            functional_test.go:555: (dbg) waiting for pods with labels "app=mysql" in namespace "default" ...
            helpers.go:247: "mysql-5787d7b5fc-kqdrj" [90a614ba-f2d8-4318-92ae-6edd7463f14c] Running
            functional_test.go:555: (dbg) pods app=mysql up and healthy within 5.005878835s
            functional_test.go:559: (dbg) Run:  kubectl --context functional-20191104T082125.872658733-18873 exec mysql-5787d7b5fc-kqdrj -- mysql -ppassword -e "show databases;"
        --- PASS: TestFunctional/parallel/AddonsCmd (0.12s)
            functional_test.go:478: (dbg) Run:  out/minikube -p functional-20191104T082125.872658733-18873 addons list
            functional_test.go:492: (dbg) Run:  out/minikube -p functional-20191104T082125.872658733-18873 addons list --format "{{.AddonName}}":"{{.AddonStatus}}"
            functional_test.go:506: (dbg) Run:  out/minikube -p functional-20191104T082125.872658733-18873 addons list -f "{{.AddonName}}":"{{.AddonStatus}}"
            functional_test.go:520: (dbg) Run:  out/minikube -p functional-20191104T082125.872658733-18873 addons list -o json
        --- PASS: TestFunctional/parallel/ProfileCmd (0.05s)
            functional_test.go:357: (dbg) Run:  out/minikube profile list
            functional_test.go:377: (dbg) Run:  out/minikube profile list --output json
        --- PASS: TestFunctional/parallel/PersistentVolumeClaim (7.75s)
            fn_pvc.go:40: (dbg) waiting for pods with labels "integration-test=storage-provisioner" in namespace "kube-system" ...
            helpers.go:247: "storage-provisioner" [6c9c2d17-8119-4170-9544-5a1cbdcc68cc] Running
            fn_pvc.go:40: (dbg) pods integration-test=storage-provisioner up and healthy within 5.010901682s
            fn_pvc.go:45: (dbg) Run:  kubectl --context functional-20191104T082125.872658733-18873 get storageclass -o=json
            fn_pvc.go:65: (dbg) Run:  kubectl --context functional-20191104T082125.872658733-18873 apply -f testdata/pvc.yaml
            fn_pvc.go:71: (dbg) Run:  kubectl --context functional-20191104T082125.872658733-18873 get pvc testpvc -o=json
            fn_pvc.go:71: (dbg) Run:  kubectl --context functional-20191104T082125.872658733-18873 get pvc testpvc -o=json
        --- PASS: TestFunctional/parallel/ServiceCmd (29.28s)
            functional_test.go:401: (dbg) Run:  kubectl --context functional-20191104T082125.872658733-18873 create deployment hello-node --image=gcr.io/hello-minikube-zero-install/hello-node
            functional_test.go:405: (dbg) Run:  kubectl --context functional-20191104T082125.872658733-18873 expose deployment hello-node --type=NodePort --port=8080
            functional_test.go:410: (dbg) waiting for pods with labels "app=hello-node" in namespace "default" ...
            helpers.go:247: "hello-node-7676b5fb8d-nxjqs" [9b3adf91-a112-4dbc-9b19-9d8801af3b43] Pending / Ready:ContainersNotReady (containers with unready status: [hello-node]) / ContainersReady:ContainersNotReady (containers with unready status: [hello-node])
            helpers.go:247: "hello-node-7676b5fb8d-nxjqs" [9b3adf91-a112-4dbc-9b19-9d8801af3b43] Running
            functional_test.go:410: (dbg) pods app=hello-node up and healthy within 27.004810384s
            functional_test.go:414: (dbg) Run:  out/minikube -p functional-20191104T082125.872658733-18873 service list
            functional_test.go:414: (dbg) Done: out/minikube -p functional-20191104T082125.872658733-18873 service list: (1.640625892s)
            functional_test.go:423: (dbg) Run:  out/minikube -p functional-20191104T082125.872658733-18873 service --namespace=default --https --url hello-node
            functional_test.go:441: (dbg) Run:  out/minikube -p functional-20191104T082125.872658733-18873 service hello-node --url --format={{.IP}}
            functional_test.go:450: (dbg) Run:  out/minikube -p functional-20191104T082125.872658733-18873 service hello-node --url
            functional_test.go:464: url: http://192.168.39.24:31849
        --- PASS: TestFunctional/parallel/DashboardCmd (41.31s)
            functional_test.go:236: (dbg) daemon: [../../out/minikube dashboard --url -p functional-20191104T082125.872658733-18873 --alsologtostderr -v=1]
            functional_test.go:241: (dbg) stopping [../../out/minikube dashboard --url -p functional-20191104T082125.872658733-18873 --alsologtostderr -v=1] ...
            helpers.go:387: unable to kill pid 21063: os: process already finished
    helpers.go:167: (dbg) Run:  out/minikube delete -p functional-20191104T082125.872658733-18873
    helpers.go:167: (dbg) Done: out/minikube delete -p functional-20191104T082125.872658733-18873: (1.601200457s)
=== RUN   TestAddons/parallel
=== RUN   TestAddons/parallel/Registry
=== PAUSE TestAddons/parallel/Registry
=== RUN   TestAddons/parallel/Ingress
=== PAUSE TestAddons/parallel/Ingress
=== CONT  TestAddons/parallel/Registry
=== CONT  TestAddons/parallel/Ingress
2019/11/04 08:27:50 [DEBUG] GET http://192.168.39.56:5000
--- PASS: TestAddons (192.32s)
    helpers.go:361: No need to wait for start slot, it is already 2019-11-04 08:24:52.30063752 +1100 AEDT m=+240.074648116
    addons_test.go:46: (dbg) Run:  out/minikube start -p addons-20191104T082452.300667929-18873 --wait=false --memory=2600 --alsologtostderr -v=1 --addons=ingress --addons=registry 
    addons_test.go:46: (dbg) Done: out/minikube start -p addons-20191104T082452.300667929-18873 --wait=false --memory=2600 --alsologtostderr -v=1 --addons=ingress --addons=registry : (2m28.528135985s)
    --- PASS: TestAddons/parallel (0.00s)
        --- PASS: TestAddons/parallel/Ingress (29.68s)
            addons_test.go:97: (dbg) waiting for pods with labels "app.kubernetes.io/name=nginx-ingress-controller" in namespace "kube-system" ...
            helpers.go:247: "nginx-ingress-controller-6fc5bcc8c9-pvwsq" [03a32eb7-f4fd-43ad-89f9-73f9c9407f1b] Running / Ready:ContainersNotReady (containers with unready status: [nginx-ingress-controller]) / ContainersReady:ContainersNotReady (containers with unready status: [nginx-ingress-controller])
            addons_test.go:97: (dbg) pods app.kubernetes.io/name=nginx-ingress-controller up and healthy within 5.005704761s
            addons_test.go:101: (dbg) Run:  kubectl --context addons-20191104T082452.300667929-18873 replace --force -f testdata/nginx-ing.yaml
            addons_test.go:101: (dbg) Done: kubectl --context addons-20191104T082452.300667929-18873 replace --force -f testdata/nginx-ing.yaml: (2.671133592s)
            addons_test.go:105: (dbg) Run:  kubectl --context addons-20191104T082452.300667929-18873 replace --force -f testdata/nginx-pod-svc.yaml
            addons_test.go:110: (dbg) waiting for pods with labels "run=nginx" in namespace "default" ...
            helpers.go:247: "nginx" [21d6e955-3c60-4673-b019-4b25d0af8f9d] Pending / Ready:ContainersNotReady (containers with unready status: [nginx]) / ContainersReady:ContainersNotReady (containers with unready status: [nginx])
            helpers.go:247: "nginx" [21d6e955-3c60-4673-b019-4b25d0af8f9d] Running
            addons_test.go:110: (dbg) pods run=nginx up and healthy within 21.024403427s
            addons_test.go:119: (dbg) Run:  out/minikube -p addons-20191104T082452.300667929-18873 ssh "curl http://127.0.0.1:80 -H 'Host: nginx.example.com'"
            addons_test.go:136: (dbg) Run:  out/minikube -p addons-20191104T082452.300667929-18873 addons disable ingress --alsologtostderr -v=1
        --- PASS: TestAddons/parallel/Registry (30.14s)
            addons_test.go:152: registry stabilized in 17.21522ms
            addons_test.go:154: (dbg) waiting for pods with labels "actual-registry=true" in namespace "kube-system" ...
            helpers.go:247: "registry-qw4s5" [3e6cc7cb-d88f-4a0c-9a83-29c7fbd01596] Running
            addons_test.go:154: (dbg) pods actual-registry=true up and healthy within 5.008192127s
            addons_test.go:157: (dbg) waiting for pods with labels "registry-proxy=true" in namespace "kube-system" ...
            helpers.go:247: "registry-proxy-dd8rs" [f07e58a3-2550-4bd1-bbbe-d0413fee18ab] Running
            addons_test.go:157: (dbg) pods registry-proxy=true up and healthy within 5.006662151s
            addons_test.go:162: (dbg) Run:  kubectl --context addons-20191104T082452.300667929-18873 delete po -l run=registry-test --now
            addons_test.go:167: (dbg) Run:  kubectl --context addons-20191104T082452.300667929-18873 run --rm registry-test --restart=Never --image=busybox -it -- sh -c "wget --spider -S http://registry.kube-system.svc.cluster.local"
            addons_test.go:167: (dbg) Done: kubectl --context addons-20191104T082452.300667929-18873 run --rm registry-test --restart=Never --image=busybox -it -- sh -c "wget --spider -S http://registry.kube-system.svc.cluster.local": (19.592473452s)
            addons_test.go:177: (dbg) Run:  out/minikube -p addons-20191104T082452.300667929-18873 ip
            addons_test.go:206: (dbg) Run:  out/minikube -p addons-20191104T082452.300667929-18873 addons disable registry --alsologtostderr -v=1
    addons_test.go:70: (dbg) Run:  out/minikube stop -p addons-20191104T082452.300667929-18873
    addons_test.go:70: (dbg) Done: out/minikube stop -p addons-20191104T082452.300667929-18873: (13.233220482s)
    addons_test.go:74: (dbg) Run:  out/minikube addons enable dashboard -p addons-20191104T082452.300667929-18873
    addons_test.go:78: (dbg) Run:  out/minikube addons disable dashboard -p addons-20191104T082452.300667929-18873
    helpers.go:167: (dbg) Run:  out/minikube delete -p addons-20191104T082452.300667929-18873
--- PASS: TestDockerFlags (235.66s)
    helpers.go:358: Waiting for start slot at 2019-11-04 08:25:52.300626185 +1100 AEDT m=+300.074636780 (sleeping 59.999921632s)  ...
    docker_test.go:42: (dbg) Run:  out/minikube start -p docker-flags-20191104T082552.300971973-18873 --wait=false --docker-env=FOO=BAR --docker-env=BAZ=BAT --docker-opt=debug --docker-opt=icc=true --alsologtostderr -v=5 
    docker_test.go:42: (dbg) Done: out/minikube start -p docker-flags-20191104T082552.300971973-18873 --wait=false --docker-env=FOO=BAR --docker-env=BAZ=BAT --docker-opt=debug --docker-opt=icc=true --alsologtostderr -v=5 : (2m54.076176539s)
    docker_test.go:47: (dbg) Run:  out/minikube -p docker-flags-20191104T082552.300971973-18873 ssh "systemctl show docker --property=Environment --no-pager"
    docker_test.go:58: (dbg) Run:  out/minikube -p docker-flags-20191104T082552.300971973-18873 ssh "systemctl show docker --property=ExecStart --no-pager"
    helpers.go:167: (dbg) Run:  out/minikube delete -p docker-flags-20191104T082552.300971973-18873
    helpers.go:167: (dbg) Done: out/minikube delete -p docker-flags-20191104T082552.300971973-18873: (1.132863792s)

--- PASS: TestVersionUpgrade (369.17s)
    helpers.go:358: Waiting for start slot at 2019-11-04 08:25:22.300626185 +1100 AEDT m=+270.074636780 (sleeping 29.999935731s)  ...
    version_upgrade_test.go:70: (dbg) Run:  /tmp/minikube-release.878849777.exe start -p vupgrade-20191104T082522.300800849-18873 --kubernetes-version=v1.11.10 --alsologtostderr -v=1 
    version_upgrade_test.go:70: (dbg) Done: /tmp/minikube-release.878849777.exe start -p vupgrade-20191104T082522.300800849-18873 --kubernetes-version=v1.11.10 --alsologtostderr -v=1 : (2m58.197200416s)
    version_upgrade_test.go:79: (dbg) Run:  /tmp/minikube-release.878849777.exe stop -p vupgrade-20191104T082522.300800849-18873
    version_upgrade_test.go:79: (dbg) Done: /tmp/minikube-release.878849777.exe stop -p vupgrade-20191104T082522.300800849-18873: (14.155612976s)
    version_upgrade_test.go:84: (dbg) Run:  /tmp/minikube-release.878849777.exe -p vupgrade-20191104T082522.300800849-18873 status --format={{.Host}}
    version_upgrade_test.go:84: (dbg) Non-zero exit: /tmp/minikube-release.878849777.exe -p vupgrade-20191104T082522.300800849-18873 status --format={{.Host}}: exit status 1 (68.698765ms)
        -- stdout --
        Stopped
        -- /stdout --
    version_upgrade_test.go:86: status error: exit status 1 (may be ok)
    helpers.go:361: No need to wait for start slot, it is already 2019-11-04 08:28:37.164720757 +1100 AEDT m=+464.938731345
    version_upgrade_test.go:95: (dbg) Run:  out/minikube start -p vupgrade-20191104T082522.300800849-18873 --kubernetes-version=v1.16.2 --alsologtostderr -v=1 
    version_upgrade_test.go:95: (dbg) Done: out/minikube start -p vupgrade-20191104T082522.300800849-18873 --kubernetes-version=v1.16.2 --alsologtostderr -v=1 : (2m23.324335645s)
    helpers.go:167: (dbg) Run:  out/minikube delete -p vupgrade-20191104T082522.300800849-18873
--- PASS: TestStartStop (626.25s)
    --- PASS: TestStartStop/group (0.00s)
        --- PASS: TestStartStop/group/docker (416.69s)
            helpers.go:358: Waiting for start slot at 2019-11-04 08:26:22.300626185 +1100 AEDT m=+330.074636780 (sleeping 1m29.999762906s)  ...
            start_stop_delete_test.go:91: (dbg) Run:  out/minikube start -p docker-20191104T082622.300761702-18873 --alsologtostderr -v=3 --wait=true --cache-images=false --kubernetes-version=v1.11.10 --kvm-network=default --kvm-qemu-uri=qemu:///system --disable-driver-mounts --keep-context=false --container-runtime=docker 
            start_stop_delete_test.go:91: (dbg) Done: out/minikube start -p docker-20191104T082622.300761702-18873 --alsologtostderr -v=3 --wait=true --cache-images=false --kubernetes-version=v1.11.10 --kvm-network=default --kvm-qemu-uri=qemu:///system --disable-driver-mounts --keep-context=false --container-runtime=docker : (3m9.827465706s)
            start_stop_delete_test.go:102: (dbg) Run:  kubectl --context docker-20191104T082622.300761702-18873 create -f testdata/busybox.yaml
            start_stop_delete_test.go:102: (dbg) Done: kubectl --context docker-20191104T082622.300761702-18873 create -f testdata/busybox.yaml: (1.045195614s)
            start_stop_delete_test.go:107: (dbg) waiting for pods with labels "integration-test=busybox" in namespace "default" ...
            helpers.go:247: "busybox" [06dd0c10-fe81-11e9-a894-5c5fdd3720f5] Pending / Ready:ContainersNotReady (containers with unready status: [busybox]) / ContainersReady:ContainersNotReady (containers with unready status: [busybox])
            helpers.go:247: "busybox" [06dd0c10-fe81-11e9-a894-5c5fdd3720f5] Running
            start_stop_delete_test.go:107: (dbg) pods integration-test=busybox up and healthy within 16.06885973s
            start_stop_delete_test.go:113: (dbg) Run:  kubectl --context docker-20191104T082622.300761702-18873 exec busybox -- /bin/sh -c "ulimit -n"
            start_stop_delete_test.go:130: (dbg) Run:  out/minikube stop -p docker-20191104T082622.300761702-18873 --alsologtostderr -v=3
            start_stop_delete_test.go:130: (dbg) Done: out/minikube stop -p docker-20191104T082622.300761702-18873 --alsologtostderr -v=3: (17.353839074s)
            start_stop_delete_test.go:135: (dbg) Run:  out/minikube status --format={{.Host}} -p docker-20191104T082622.300761702-18873
            start_stop_delete_test.go:135: (dbg) Non-zero exit: out/minikube status --format={{.Host}} -p docker-20191104T082622.300761702-18873: exit status 1 (37.663869ms)
                -- stdout --
                Stopped
                -- /stdout --
            start_stop_delete_test.go:135: status error: exit status 1 (may be ok)
            helpers.go:361: No need to wait for start slot, it is already 2019-11-04 08:30:07.029124735 +1100 AEDT m=+554.803135321
            start_stop_delete_test.go:141: (dbg) Run:  out/minikube start -p docker-20191104T082622.300761702-18873 --alsologtostderr -v=3 --wait=true --cache-images=false --kubernetes-version=v1.11.10 --kvm-network=default --kvm-qemu-uri=qemu:///system --disable-driver-mounts --keep-context=false --container-runtime=docker 
            start_stop_delete_test.go:141: (dbg) Done: out/minikube start -p docker-20191104T082622.300761702-18873 --alsologtostderr -v=3 --wait=true --cache-images=false --kubernetes-version=v1.11.10 --kvm-network=default --kvm-qemu-uri=qemu:///system --disable-driver-mounts --keep-context=false --container-runtime=docker : (1m35.29120763s)
            start_stop_delete_test.go:149: (dbg) waiting for pods with labels "integration-test=busybox" in namespace "default" ...
            helpers.go:247: "busybox" [06dd0c10-fe81-11e9-a894-5c5fdd3720f5] Running
            start_stop_delete_test.go:149: (dbg) pods integration-test=busybox up and healthy within 5.642024689s
            start_stop_delete_test.go:153: (dbg) Run:  out/minikube status --format={{.Host}} -p docker-20191104T082622.300761702-18873
            helpers.go:167: (dbg) Run:  out/minikube delete -p docker-20191104T082622.300761702-18873
        --- PASS: TestStartStop/group/cni (467.93s)
            helpers.go:358: Waiting for start slot at 2019-11-04 08:27:52.300626185 +1100 AEDT m=+420.074636780 (sleeping 2m59.999736017s)  ...
            start_stop_delete_test.go:91: (dbg) Run:  out/minikube start -p cni-20191104T082752.300771556-18873 --alsologtostderr -v=3 --wait=true --feature-gates ServerSideApply=true --network-plugin=cni --extra-config=kubelet.network-plugin=cni --extra-config=kubeadm.pod-network-cidr=192.168.111.111/16 --kubernetes-version=v1.16.2 
            start_stop_delete_test.go:91: (dbg) Done: out/minikube start -p cni-20191104T082752.300771556-18873 --alsologtostderr -v=3 --wait=true --feature-gates ServerSideApply=true --network-plugin=cni --extra-config=kubelet.network-plugin=cni --extra-config=kubeadm.pod-network-cidr=192.168.111.111/16 --kubernetes-version=v1.16.2 : (3m50.111617403s)
            start_stop_delete_test.go:99: WARNING: cni mode requires additional setup before pods can schedule :(
            start_stop_delete_test.go:130: (dbg) Run:  out/minikube stop -p cni-20191104T082752.300771556-18873 --alsologtostderr -v=3
            start_stop_delete_test.go:130: (dbg) Done: out/minikube stop -p cni-20191104T082752.300771556-18873 --alsologtostderr -v=3: (13.361886938s)
            start_stop_delete_test.go:135: (dbg) Run:  out/minikube status --format={{.Host}} -p cni-20191104T082752.300771556-18873
            start_stop_delete_test.go:135: (dbg) Non-zero exit: out/minikube status --format={{.Host}} -p cni-20191104T082752.300771556-18873: exit status 1 (40.269985ms)
                -- stdout --
                Stopped
                -- /stdout --
            start_stop_delete_test.go:135: status error: exit status 1 (may be ok)
            helpers.go:361: No need to wait for start slot, it is already 2019-11-04 08:31:55.814766655 +1100 AEDT m=+663.588777252
            start_stop_delete_test.go:141: (dbg) Run:  out/minikube start -p cni-20191104T082752.300771556-18873 --alsologtostderr -v=3 --wait=true --feature-gates ServerSideApply=true --network-plugin=cni --extra-config=kubelet.network-plugin=cni --extra-config=kubeadm.pod-network-cidr=192.168.111.111/16 --kubernetes-version=v1.16.2 
            start_stop_delete_test.go:141: (dbg) Done: out/minikube start -p cni-20191104T082752.300771556-18873 --alsologtostderr -v=3 --wait=true --feature-gates ServerSideApply=true --network-plugin=cni --extra-config=kubelet.network-plugin=cni --extra-config=kubeadm.pod-network-cidr=192.168.111.111/16 --kubernetes-version=v1.16.2 : (43.328528439s)
            start_stop_delete_test.go:148: WARNING: cni mode requires additional setup before pods can schedule :(
            start_stop_delete_test.go:153: (dbg) Run:  out/minikube status --format={{.Host}} -p cni-20191104T082752.300771556-18873
            helpers.go:167: (dbg) Run:  out/minikube delete -p cni-20191104T082752.300771556-18873
        --- PASS: TestStartStop/group/containerd (555.16s)
            helpers.go:358: Waiting for start slot at 2019-11-04 08:26:52.300626185 +1100 AEDT m=+360.074636780 (sleeping 1m59.999750306s)  ...
            start_stop_delete_test.go:91: (dbg) Run:  out/minikube start -p containerd-20191104T082652.300780979-18873 --alsologtostderr -v=3 --wait=true --container-runtime=containerd --docker-opt containerd=/var/run/containerd/containerd.sock --apiserver-port=8444 
            start_stop_delete_test.go:91: (dbg) Done: out/minikube start -p containerd-20191104T082652.300780979-18873 --alsologtostderr -v=3 --wait=true --container-runtime=containerd --docker-opt containerd=/var/run/containerd/containerd.sock --apiserver-port=8444 : (4m29.732924894s)
            start_stop_delete_test.go:102: (dbg) Run:  kubectl --context containerd-20191104T082652.300780979-18873 create -f testdata/busybox.yaml
            start_stop_delete_test.go:102: (dbg) Done: kubectl --context containerd-20191104T082652.300780979-18873 create -f testdata/busybox.yaml: (3.022589629s)
            start_stop_delete_test.go:107: (dbg) waiting for pods with labels "integration-test=busybox" in namespace "default" ...
            helpers.go:247: "busybox" [90846118-a891-4edd-86a1-4df14dbdacdd] Pending
            helpers.go:247: "busybox" [90846118-a891-4edd-86a1-4df14dbdacdd] Pending / Ready:ContainersNotReady (containers with unready status: [busybox]) / ContainersReady:ContainersNotReady (containers with unready status: [busybox])
            helpers.go:247: "busybox" [90846118-a891-4edd-86a1-4df14dbdacdd] Running
            start_stop_delete_test.go:107: (dbg) pods integration-test=busybox up and healthy within 11.883410366s
            start_stop_delete_test.go:113: (dbg) Run:  kubectl --context containerd-20191104T082652.300780979-18873 exec busybox -- /bin/sh -c "ulimit -n"
            start_stop_delete_test.go:130: (dbg) Run:  out/minikube stop -p containerd-20191104T082652.300780979-18873 --alsologtostderr -v=3
            start_stop_delete_test.go:130: (dbg) Done: out/minikube stop -p containerd-20191104T082652.300780979-18873 --alsologtostderr -v=3: (1m31.837876123s)
            start_stop_delete_test.go:135: (dbg) Run:  out/minikube status --format={{.Host}} -p containerd-20191104T082652.300780979-18873
            start_stop_delete_test.go:135: (dbg) Non-zero exit: out/minikube status --format={{.Host}} -p containerd-20191104T082652.300780979-18873: exit status 1 (35.925354ms)
                -- stdout --
                Stopped
                -- /stdout --
            start_stop_delete_test.go:135: status error: exit status 1 (may be ok)
            helpers.go:361: No need to wait for start slot, it is already 2019-11-04 08:33:09.026967426 +1100 AEDT m=+736.800978015
            start_stop_delete_test.go:141: (dbg) Run:  out/minikube start -p containerd-20191104T082652.300780979-18873 --alsologtostderr -v=3 --wait=true --container-runtime=containerd --docker-opt containerd=/var/run/containerd/containerd.sock --apiserver-port=8444 
            start_stop_delete_test.go:141: (dbg) Done: out/minikube start -p containerd-20191104T082652.300780979-18873 --alsologtostderr -v=3 --wait=true --container-runtime=containerd --docker-opt containerd=/var/run/containerd/containerd.sock --apiserver-port=8444 : (52.484285038s)
            start_stop_delete_test.go:149: (dbg) waiting for pods with labels "integration-test=busybox" in namespace "default" ...
            helpers.go:247: "busybox" [90846118-a891-4edd-86a1-4df14dbdacdd] Running / Ready:ContainersNotReady (containers with unready status: [busybox]) / ContainersReady:ContainersNotReady (containers with unready status: [busybox])
            helpers.go:247: "busybox" [90846118-a891-4edd-86a1-4df14dbdacdd] Running
            start_stop_delete_test.go:149: (dbg) pods integration-test=busybox up and healthy within 5.008644281s
            start_stop_delete_test.go:153: (dbg) Run:  out/minikube status --format={{.Host}} -p containerd-20191104T082652.300780979-18873
            helpers.go:167: (dbg) Run:  out/minikube delete -p containerd-20191104T082652.300780979-18873
        --- PASS: TestStartStop/group/crio (626.25s)
            helpers.go:358: Waiting for start slot at 2019-11-04 08:27:22.300626185 +1100 AEDT m=+390.074636780 (sleeping 2m29.999745542s)  ...
            start_stop_delete_test.go:91: (dbg) Run:  out/minikube start -p crio-20191104T082722.304206154-18873 --alsologtostderr -v=3 --wait=true --container-runtime=crio --disable-driver-mounts --extra-config=kubeadm.ignore-preflight-errors=SystemVerification 
            start_stop_delete_test.go:91: (dbg) Done: out/minikube start -p crio-20191104T082722.304206154-18873 --alsologtostderr -v=3 --wait=true --container-runtime=crio --disable-driver-mounts --extra-config=kubeadm.ignore-preflight-errors=SystemVerification : (4m52.989940404s)
            start_stop_delete_test.go:102: (dbg) Run:  kubectl --context crio-20191104T082722.304206154-18873 create -f testdata/busybox.yaml
            start_stop_delete_test.go:107: (dbg) waiting for pods with labels "integration-test=busybox" in namespace "default" ...
            helpers.go:247: "busybox" [2660e105-871f-4b5e-8e06-306ee3ed6648] Pending
            helpers.go:247: "busybox" [2660e105-871f-4b5e-8e06-306ee3ed6648] Pending / Ready:ContainersNotReady (containers with unready status: [busybox]) / ContainersReady:ContainersNotReady (containers with unready status: [busybox])
            helpers.go:247: "busybox" [2660e105-871f-4b5e-8e06-306ee3ed6648] Running
            start_stop_delete_test.go:107: (dbg) pods integration-test=busybox up and healthy within 8.510315758s
            start_stop_delete_test.go:113: (dbg) Run:  kubectl --context crio-20191104T082722.304206154-18873 exec busybox -- /bin/sh -c "ulimit -n"
            start_stop_delete_test.go:130: (dbg) Run:  out/minikube stop -p crio-20191104T082722.304206154-18873 --alsologtostderr -v=3
            start_stop_delete_test.go:130: (dbg) Done: out/minikube stop -p crio-20191104T082722.304206154-18873 --alsologtostderr -v=3: (1m32.087370333s)
            start_stop_delete_test.go:135: (dbg) Run:  out/minikube status --format={{.Host}} -p crio-20191104T082722.304206154-18873
            start_stop_delete_test.go:135: (dbg) Non-zero exit: out/minikube status --format={{.Host}} -p crio-20191104T082722.304206154-18873: exit status 1 (35.519744ms)
                -- stdout --
                Stopped
                -- /stdout --
            start_stop_delete_test.go:135: status error: exit status 1 (may be ok)
            helpers.go:361: No need to wait for start slot, it is already 2019-11-04 08:33:56.977436358 +1100 AEDT m=+784.751446950
            start_stop_delete_test.go:141: (dbg) Run:  out/minikube start -p crio-20191104T082722.304206154-18873 --alsologtostderr -v=3 --wait=true --container-runtime=crio --disable-driver-mounts --extra-config=kubeadm.ignore-preflight-errors=SystemVerification 
            start_stop_delete_test.go:141: (dbg) Done: out/minikube start -p crio-20191104T082722.304206154-18873 --alsologtostderr -v=3 --wait=true --container-runtime=crio --disable-driver-mounts --extra-config=kubeadm.ignore-preflight-errors=SystemVerification : (1m15.390161848s)
            start_stop_delete_test.go:149: (dbg) waiting for pods with labels "integration-test=busybox" in namespace "default" ...
            helpers.go:247: "busybox" [2660e105-871f-4b5e-8e06-306ee3ed6648] Running
            start_stop_delete_test.go:149: (dbg) pods integration-test=busybox up and healthy within 5.011162349s
            start_stop_delete_test.go:153: (dbg) Run:  out/minikube status --format={{.Host}} -p crio-20191104T082722.304206154-18873
            helpers.go:167: (dbg) Run:  out/minikube delete -p crio-20191104T082722.304206154-18873
PASS
ok  	k8s.io/minikube/test/integration	866.327s
{{< /highlight >}}

