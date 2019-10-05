---
date: "2019-09-17"
title: "Inside minikube source code"
author : "Nanik Tolaram (nanikjava@gmail.com)" 
---
**NOTE: The source code discussed in this article is based on _master_ branch**

We are going to create a separate brand new GOPATH directory for minikube to demonstrate how to do the whole process fresh from scratch. On my local machine I've create a directory called *minikubegopath* with the following structure 

{{< highlight text >}}

.
├── bin
├── pkg
└── src
    └── k8s.io
{{< /highlight >}}


Make sure are you inside the src/k8s.io directory when cloning minikube

{{< highlight text >}}
git clone https://github.com/kubernetes/minikube
{{< /highlight >}}

Open the minikube inside the IDE and set the GOPATH to the new GOPATH directory that have minikube, setup it as shown in the screenshot below

![MinikubeSourceCodeSettingGoPath](/media/minikubesourcecode/minikubesourcecode_setupgopath_ide.png)

Select the the *main.go* file that reside in cmd/minikube directory and hit the run button. You will get an error output as shown below, this is because there are some files that need to be generated and this can be done using the *make* command

![MinikubeSourceCodeHitRun](/media/minikubesourcecode/minikubesourcecode_hitrun.png)


{{< highlight text >}}
go: downloading github.com/mattn/go-isatty v0.0.8
go: downloading github.com/pkg/profile v0.0.0-20161223203901-3a8809bd8a80
go: downloading github.com/golang/glog v0.0.0-20160126235308-23def4e6c14b

.....
.....
.....


# k8s.io/minikube/pkg/minikube/translate
../../pkg/minikube/translate/translate.go:77:12: undefined: Asset

Compilation finished with exit code 2
{{< /highlight >}}

Use the following *make* command to generate the files needed

{{< highlight text >}}
make pkg/minikube/assets/assets.go pkg/minikube/translate/translations.go
{{< /highlight >}}

You will see the following output

{{< highlight text >}}
which go-bindata || GO111MODULE=off GOBIN=XXXXXXXX/minikubegopath/bin go get github.com/jteeuwen/go-bindata/...
....... go-bindata -nomemcopy -o pkg/minikube/assets/assets.go -pkg assets deploy/addons/...
gofmt -s -w pkg/minikube/assets/assets.go
......
GOOS="linux" GOARCH="amd64" go build -tags "container_image_ostree_stub containers_image_openpgp" -ldflags="-X k8s.io/minikube/pkg/version.version=v1.4.0-beta.2 -X k8s.io/minikube/pkg/version.isoVersion=v1.4.0-beta.2 -X k8s.io/minikube/pkg/version.isoPath=minikube/iso -X k8s.io/minikube/pkg/version.gitCommitID="d1e468085d9af12e5d130a6cb3b2186a5db87a0e-dirty"" -a -o out/minikube-linux-amd64 k8s.io/minikube/cmd/minikube
{{< /highlight >}}


Following are the generated files

{{< highlight text >}}
pkg/minikube/assets/assets.go --> generate source code to allow loading of addons .yaml files
pkg/minikube/translate/translations.go --> generate source code for translations data file
{{< /highlight >}}

<h1>Makefile</h1>

Following table explains some of the available tasks:

{{< bootstrap-table "table table-responsive table-dark table-danger" >}}
|      |
|------|
|**pkg/minikube/assets/assets.go**  ==> generate source code needed to build minikube. Source code generated are used for loading of .yaml file |
|**pkg/minikube/translate/translations.go** ==> generate source code for translations data file |
|**minikube-linux-amd64** ==> generate minikube for Linux 64-bit. Windows task minikube-windows-amd64.exe and MacOS minikube-darwin-amd64  |
|**out/e2e-linux-amd64** ==> compile end-to-end testing and minikube binary for testing purposes|
|**minikube_iso** ==> used internally to generate the .iso file. Better use the out/minikube.iso |
|**iso-menuconfig** ==> change buildroot configuration for  minikube iso|
|**linux-menuconfig** ==> change kernel configuration for minikube iso|
|**all** ==> compile and generate minikube for the 3 platforms - Linux, Windows and MacOS  |
|**cross** ==> build minikube for the 3 platforms - Linux, Windows and MacOS |

{{< /bootstrap-table >}}


<h1>Source structure</h1>


{{< highlight text >}}
.
├── cmd
│   ├── drivers
│   │   ├── hyperkit
│   │   └── kvm
│   ├── extract
│   ├── gvisor
│   ├── minikube
│   │   └── cmd
│   │       └── config
│   └── storage-provisioner
├── deploy
│   ├── addons
│   │   ├── dashboard
│   │   ├── efk
│   │   ├── freshpod
│   │   ├── gpu
│   │   ├── gvisor
│   │   ├── heapster
│   │   ├── ingress
│   │   ├── logviewer
│   │   ├── metrics-server
│   │   ├── registry
│   │   ├── registry-creds
│   │   ├── storageclass
│   │   ├── storage-provisioner
│   │   └── storage-provisioner-gluster
│   ├── gvisor
│   ├── iso
│   │   └── minikube-iso
│   │       ├── board
│   │       │   └── coreos
│   │       │       └── minikube
│   │       │           └── rootfs-overlay
│   │       │               ├── etc
│   │       │               │   ├── cni
│   │       │               │   │   └── net.d
│   │       │               │   ├── docker
│   │       │               │   ├── ssh
│   │       │               │   ├── systemd
│   │       │               │   │   ├── network
│   │       │               │   │   └── system
│   │       │               │   │       ├── getty.target.wants
│   │       │               │   │       └── systemd-timesyncd.service.d
│   │       │               │   └── udev
│   │       │               │       └── rules.d
│   │       │               ├── usr
│   │       │               │   └── bin
│   │       │               └── var
│   │       │                   ├── lib
│   │       │                   │   └── boot2docker
│   │       │                   ├── log
│   │       │                   └── run
│   │       │                       └── crio
│   │       ├── configs
│   │       └── package
│   │           ├── automount
│   │           ├── cni-bin
│   │           ├── cni-plugins-bin
│   │           ├── conmon-master
│   │           ├── containerd-bin
│   │           ├── crictl-bin
│   │           ├── crio-bin
│   │           ├── docker-bin
│   │           ├── gluster
│   │           ├── hyperv-daemons
│   │           ├── podman
│   │           ├── runc-master
│   │           └── vbox-guest
│   ├── minikube
│   └── storage-provisioner
├── docs
│   └── contributors
├── hack
│   ├── boilerplate
│   ├── help_text
│   ├── jenkins
│   │   └── installers
│   ├── kubernetes_version
│   ├── prow
│   └── release_notes
├── images
│   └── logo
├── installers
│   ├── darwin
│   │   └── brew-cask
│   ├── linux
│   │   ├── archlinux
│   │   ├── archlinux-driver
│   │   ├── deb
│   │   │   ├── kvm2_deb_template
│   │   │   │   └── DEBIAN
│   │   │   └── minikube_deb_template
│   │   │       └── DEBIAN
│   │   ├── kvm
│   │   └── rpm
│   │       ├── kvm2_rpm_template
│   │       └── minikube_rpm_template
│   └── windows
├── pkg
│   ├── drivers
│   │   ├── hyperkit
│   │   ├── kvm
│   │   └── none
│   ├── gvisor
│   ├── initflag
│   ├── kapi
│   ├── minikube
│   │   ├── assets
│   │   ├── bootstrapper
│   │   │   └── kubeadm
│   │   │       └── testdata
│   │   │           ├── v1.11
│   │   │           ├── v1.12
│   │   │           ├── v1.13
│   │   │           ├── v1.14
│   │   │           ├── v1.15
│   │   │           └── v1.16
│   │   ├── cluster
│   │   ├── command
│   │   ├── config
│   │   │   └── testdata
│   │   ├── constants
│   │   ├── cruntime
│   │   ├── drivers
│   │   │   ├── hyperkit
│   │   │   ├── hyperv
│   │   │   ├── kvm2
│   │   │   ├── none
│   │   │   ├── parallels
│   │   │   ├── virtualbox
│   │   │   ├── vmware
│   │   │   └── vmwarefusion
│   │   ├── exit
│   │   ├── extract
│   │   │   └── testdata
│   │   ├── kubeconfig
│   │   │   └── testdata
│   │   │       └── kubeconfig
│   │   ├── logs
│   │   ├── machine
│   │   ├── notify
│   │   ├── out
│   │   ├── problem
│   │   ├── proxy
│   │   ├── registry
│   │   ├── service
│   │   ├── sshutil
│   │   ├── storageclass
│   │   ├── tests
│   │   ├── translate
│   │   └── tunnel
│   ├── provision
│   ├── storage
│   ├── util
│   │   ├── lock
│   │   └── retry
│   └── version
├── site
│   ├── assets
│   │   ├── icons
│   │   └── scss
│   ├── content
│   │   └── en
│   │       ├── blog
│   │       │   ├── news
│   │       │   └── releases
│   │       ├── community
│   │       └── docs
│   │           ├── Concepts
│   │           ├── Contributing
│   │           ├── Examples
│   │           ├── Overview
│   │           ├── Reference
│   │           │   ├── Commands
│   │           │   ├── Configuration
│   │           │   ├── Drivers
│   │           │   │   └── includes
│   │           │   └── Networking
│   │           ├── Start
│   │           │   └── includes
│   │           ├── Tasks
│   │           │   └── Registry
│   │           └── Tutorials
│   ├── layouts
│   │   ├── partials
│   │   │   └── hooks
│   │   └── shortcodes
│   ├── static
│   │   ├── css
│   │   └── js
│   └── themes
│       └── docsy
├── test
│   └── integration
│       └── testdata
│           ├── kvm2-driver-older-version
│           └── kvm2-driver-without-version
├── third_party
│   └── go9p
│       └── ufs
└── translations

{{< /highlight >}}

Following table are description of the different directories of minikube's source code. **Note: This is by no means a complete list of the source directory structure as master branch do change from time to time**

{{< bootstrap-table "table table-responsive table-dark table-striped table-bordered" >}}
|Directory Name    |Description                                      |
|------------------|-------------------------------------------------|
|**drivers**           |docker-machine plugin code (libvirt and hyperkit)|
|**cmd/gvisor**           |Code to start gvisor and do all the necessary steps to restart the dependencies (containerd, etc)
|**minikube**          | minikube cli handling code |
|**storage-provisioner**|Main function to trigger the storage provisioner module|
|**deploy**          | Contains files that are needed to build minikube inside docker |
|**deploy/addons**          | Lots of minikube addons (*.yaml) inside this directory such as: gvisor → used to securely run pods with untrusted workloads, freshpods → restart container,etc  |
|**deploy/gvisor**      | contain Dockerfile to run gvisor-addon  |
|**deploy/iso**          | Contains documentation, packages and configuration for building minikube iso. Some of the packages that are included – automount,containerd and many more. It also contains configuration file for building buildroot. Pretty much anything related to building iso in inside this directory |
|**deploy/minikube**          | Contains few json files related to release |
|**deploy/storage-provisioner**            | Contain dockerfile to run storage-provisioner|
|**hack/jenkins**          | Jenkins configuration file|
|**hack/kubernetes_version**               | Tool to generate Minikube version number|
|**hack/release_notes**          |Code to create release notes |
|**installers/darwin**           | script to create Darwin installer |
|**installers/linux**          | Linux – deb, archlinux, rpm, etc |
|**installers/windows**               | Windows installers |
|**pkg/drivers**          | Driver related code – kvm, hyperkit|
|**site**          |Docsy template for generating documents as website |
|**test**     |  Integration test for Minikube |
|**go9p**            |  Go implementation of 9P2000 protocol (9P (or the Plan 9 Filesystem Protocol or Styx) is a network protocol developed for the Plan 9 from Bell Labs distributed operating system as the means of connecting the components of a Plan 9 system. Files are key objects in Plan 9. They represent windows, network connections, processes, and almost anything else available in the operating system. )|
|**translations**  | Language translation at the moment CN and FR|
{{< /bootstrap-table >}}
