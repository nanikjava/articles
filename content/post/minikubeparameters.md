---
date: "2019-09-01"
title: "Minikube parameters (Work in progress)"
author : "Nanik Tolaram (nanikjava@gmail.com)" 

---

Following are the commands available for minikube. 

{{< highlight html >}}

Basic Commands:
  start          Starts a local kubernetes cluster
  status         Gets the status of a local kubernetes cluster
  stop           Stops a running local kubernetes cluster
  delete         Deletes a local kubernetes cluster
  dashboard      Access the kubernetes dashboard running within the minikube cluster

Images Commands:
  docker-env     Sets up docker env variables; similar to '$(docker-machine env)'
  cache          Add or delete an image from the local cache.

Configuration and Management Commands:
  addons         Modify minikube's kubernetes addons
  config         Modify minikube config
  profile        Profile gets or sets the current minikube profile
  update-context Verify the IP address of the running cluster in kubeconfig.

Networking and Connectivity Commands:
  service        Gets the kubernetes URL(s) for the specified service in your local cluster
  tunnel         tunnel makes services of type LoadBalancer accessible on localhost

Advanced Commands:
  mount          Mounts the specified directory into minikube
  ssh            Log into or run a command on a machine with SSH; similar to 'docker-machine ssh'
  kubectl        Run kubectl

Troubleshooting Commands:
  ssh-key        Retrieve the ssh identity key path of the specified cluster
  ip             Retrieves the IP address of the running cluster
  logs           Gets the logs of the running instance, used for debugging minikube, not user code.
  update-check   Print current and latest version number
  version        Print the version of minikube

Other Commands:
  completion     Outputs minikube shell completion for the given shell (bash or zsh)
{{< /highlight  >}}

The source code for the configurations outlined above can be found inside *cmd/minikube/cmd* directory. Noticed how the .go files are named the same as the configuration parameter which makes it easier for identifying the relevant code.

{{< highlight html >}}
├── cache.go
├── cache_list.go
├── completion.go
├── dashboard.go
├── delete.go
├── env.go
├── env_test.go
├── generate-docs.go
├── ip.go
├── kubectl.go
├── logs.go
├── mount.go
├── root.go
├── root_test.go
├── service.go
├── service_list.go
├── ssh.go
├── ssh-key.go
├── start.go
├── start_test.go
├── status.go
├── stop.go
├── tunnel.go
├── update-check.go
├── update-context.go
└── version.go
{{< /highlight  >}}

The main minikube entry point reside inside *cmd/minikube* directory in a file called **main.go**

{{< highlight go >}}
package main

import (
	"bytes"
	"fmt"
	"log"
	"os"
	"regexp"
	"strconv"

	// initflag must be imported before any other minikube pkg.
	// Fix for https://github.com/kubernetes/minikube/issues/4866
	_ "k8s.io/minikube/pkg/initflag"

	mlog "github.com/docker/machine/libmachine/log"

	....
	....
	....
	....
	_ "k8s.io/minikube/pkg/provision"
)

const minikubeEnableProfile = "MINIKUBE_ENABLE_PROFILING"

var (
	machineLogErrorRe   = regexp.MustCompile(`(?i) (failed|error|fatal)`)
	machineLogWarningRe = regexp.MustCompile(`(?i)warning`)
)

func main() {
	bridgeLogMessages()
	defer glog.Flush()

	if os.Getenv(minikubeEnableProfile) == "1" {
		defer profile.Start(profile.TraceProfile).Stop()
	}
	if os.Getenv(constants.IsMinikubeChildProcess) == "" {
		machine.StartDriver()
	}
	out.SetOutFile(os.Stdout)
	out.SetErrFile(os.Stderr)
	cmd.Execute()
}

....
....
....
....

{{< /highlight  >}}






<h1>start</h1>

Command to trigger the minikube starting process. There are a variety of options that you can pass into minikube

--cache-images=true --> cache docker images that will be used by minikube

{{< highlight html >}}
      --apiserver-ips=[]: A set of apiserver IP Addresses which are used in the generated certificate for kubernetes.  This can be used if you want to make the apiserver available from outside the machine
      --apiserver-name='minikubeCA': The apiserver name which is used in the generated certificate for kubernetes.  This can be used if you want to make the apiserver available from outside the machine
      --apiserver-names=[]: A set of apiserver names which are used in the generated certificate for kubernetes.  This can be used if you want to make the apiserver available from outside the machine
      --apiserver-port=8443: The apiserver listening port
      --cache-images=true: If true, cache docker images for the current bootstrapper and load them into the machine. Always false with --vm-driver=none.
      --container-runtime='docker': The container runtime to be used (docker, crio, containerd).
      --cpus=2: Number of CPUs allocated to the minikube VM.
      --cri-socket='': The cri socket path to be used.
      --disable-driver-mounts=false: Disables the filesystem mounts provided by the hypervisors
      --disk-size='20000mb': Disk size allocated to the minikube VM (format: <number>[<unit>], where unit = b, k, m or g).
      --dns-domain='cluster.local': The cluster dns domain name used in the kubernetes cluster
      --dns-proxy=false: Enable proxy for NAT DNS requests (virtualbox driver only)
      --docker-env=[]: Environment variables to pass to the Docker daemon. (format: key=value)
      --docker-opt=[]: Specify arbitrary flags to pass to the Docker daemon. (format: key=value)
      --download-only=false: If true, only download and cache files for later use - don't install or start anything.
      --embed-certs=false: if true, will embed the certs in kubeconfig.
      --enable-default-cni=false: Enable the default CNI plugin (/etc/cni/net.d/k8s.conf). Used in conjunction with "--network-plugin=cni".
      --extra-config=: A set of key=value pairs that describe configuration that may be passed to different components.
                The key should be '.' separated, and the first part before the dot is the component to apply the configuration to.
                Valid components are: kubelet, kubeadm, apiserver, controller-manager, etcd, proxy, scheduler
                Valid kubeadm parameters: ignore-preflight-errors, dry-run, kubeconfig, kubeconfig-dir, node-name, cri-socket, experimental-upload-certs, certificate-key, rootfs, pod-network-cidr
      --feature-gates='': A set of key=value pairs that describe feature gates for alpha/experimental features.
      --force=false: Force minikube to perform possibly dangerous operations
      --host-dns-resolver=true: Enable host resolver for NAT DNS requests (virtualbox driver only)
      --host-only-cidr='192.168.99.1/24': The CIDR to be used for the minikube VM (virtualbox driver only)
      --hyperkit-vpnkit-sock='': Location of the VPNKit socket used for networking. If empty, disables Hyperkit VPNKitSock, if 'auto' uses Docker for Mac VPNKit connection, otherwise uses the specified VSock (hyperkit driver only)
      --hyperkit-vsock-ports=[]: List of guest VSock ports that should be exposed as sockets on the host (hyperkit driver only)
      --hyperv-virtual-switch='': The hyperv virtual switch name. Defaults to first found. (hyperv driver only)
      --image-mirror-country='': Country code of the image mirror to be used. Leave empty to use the global one. For Chinese mainland users, set it to cn.
      --image-repository='': Alternative image repository to pull docker images from. This can be used when you have limited access to gcr.io. Set it to "auto" to let minikube decide one for you. For Chinese mainland users, you may use local gcr.io mirrors such as registry.cn-hangzhou.aliyuncs.com/google_containers
      --insecure-registry=[]: Insecure Docker registries to pass to the Docker daemon.  The default service CIDR range will automatically be added.
      --iso-url='https://storage.googleapis.com/minikube/iso/minikube-v0.0.0-unset.iso': Location of the minikube iso.
      --keep-context=false: This will keep the existing kubectl context and will create a minikube context.
      --kubernetes-version='v1.16.0-rc.2': The kubernetes version that the minikube VM will use (ex: v1.2.3)
      --kvm-gpu=false: Enable experimental NVIDIA GPU support in minikube
      --kvm-hidden=false: Hide the hypervisor signature from the guest in minikube (kvm2 driver only)
      --kvm-network='default': The KVM network name. (kvm2 driver only)
      --kvm-qemu-uri='qemu:///system': The KVM QEMU connection URI. (kvm2 driver only)
      --memory='2000mb': Amount of RAM allocated to the minikube VM (format: <number>[<unit>], where unit = b, k, m or g).
      --mount=false: This will start the mount daemon and automatically mount files into minikube.
      --mount-string='/home/nanik:/minikube-host': The argument to pass the minikube mount command on start.
      --native-ssh=true: Use native Golang SSH client (default true). Set to 'false' to use the command line 'ssh' command when accessing the docker machine. Useful for the machine drivers when they will not start with 'Waiting for SSH'.
      --network-plugin='': The name of the network plugin.
      --nfs-share=[]: Local folders to share with Guest via NFS mounts (hyperkit driver only)
      --nfs-shares-root='/nfsshares': Where to root the NFS Shares, defaults to /nfsshares (hyperkit driver only)
      --no-vtx-check=false: Disable checking for the availability of hardware virtualization before the vm is started (virtualbox driver only)
      --registry-mirror=[]: Registry mirrors to pass to the Docker daemon
      --service-cluster-ip-range='10.96.0.0/12': The CIDR to be used for service cluster IPs.
      --uuid='': Provide VM UUID to restore MAC address (hyperkit driver only)
      --vm-driver='': Driver is one of: [virtualbox parallels vmwarefusion kvm2 vmware none] (defaults to virtualbox)
      --wait=true: Wait until Kubernetes core services are healthy before exiting.
      --wait-timeout=6m0s: max time to wait per Kubernetes core services to be healthy.
{{< /highlight  >}}

The *start* option source code detecting the different options pass resides inside **cmd/minikube/cmd/start.go**

{{< bootstrap-table "table table-responsive table-dark table-danger" >}}
|      |
|------|
|**--enable-default-cni** --  using this flag it will use the default cni config inside *pkg/minikube/bootstrapper/kubeadm/default_cni.go* which will be stored inside /etc/cni/net.d directory as k8s.conf


{{< /bootstrap-table >}}




<h1>status</h1>

{{< highlight html >}}

Gets the status of a local kubernetes cluster.
        Exit status contains the status of minikube's VM, cluster and kubernetes encoded on it's bits in this order from right
to left.
        Eg: 7 meaning: 1 (for minikube NOK) + 2 (for cluster NOK) + 4 (for kubernetes NOK)

Options:
      --format='host: {{.Host}}
kubelet: {{.Kubelet}}
apiserver: {{.APIServer}}
kubectl: {{.Kubeconfig}}
': Go template format string for the status output.  The format for Go templates can be found here:
https://golang.org/pkg/text/template/
For the list accessible variables for the template, see the struct values here:
https://godoc.org/k8s.io/minikube/cmd/minikube/cmd#Status
{{< /highlight  >}}


<h1>cache</h1>


{{< highlight html >}}

Add or delete an image from the local cache.

Available Commands:
  add         Add an image to local cache.
  delete      Delete an image from the local cache.
  list        List all available images from the local cache.

{{< /highlight  >}}


<h1>addons</h1>

{{< highlight html >}}

addons modifies minikube addons files using subcommands like "minikube addons enable heapster"

Available Commands:
  configure   Configures the addon w/ADDON_NAME within minikube (example: minikube addons configure registry-creds). For
a list of available addons use: minikube addons list 
  disable     Disables the addon w/ADDON_NAME within minikube (example: minikube addons disable dashboard). For a list
of available addons use: minikube addons list 
  enable      Enables the addon w/ADDON_NAME within minikube (example: minikube addons enable dashboard). For a list of
available addons use: minikube addons list 
  list        Lists all available minikube addons as well as their current statuses (enabled/disabled)
  open        Opens the addon w/ADDON_NAME within minikube (example: minikube addons open dashboard). For a list of
available addons use: minikube addons list 

Options:
      --format='- {{.AddonName}}: {{.AddonStatus}}
': Go template format string for the addon list output.  The format for Go templates can be found here:
https://golang.org/pkg/text/template/
For the list of accessible variables for the template, see the struct values here:
https://godoc.org/k8s.io/minikube/cmd/minikube/cmd/config#AddonListTemplate

{{< /highlight  >}}



<h1>config</h1>

{{< highlight html >}}

config modifies minikube config files using subcommands like "minikube config set vm-driver kvm"
Configurable fields: 

 * vm-driver
 * container-runtime
 * feature-gates
 * v
 * cpus
 * disk-size
 * host-only-cidr
 * memory
 * log_dir
 * kubernetes-version
 * iso-url
 * WantUpdateNotification
 * ReminderWaitPeriodInHours
 * WantReportError
 * WantReportErrorPrompt
 * WantKubectlDownloadMsg
 * WantNoneDriverWarning
 * profile
 * bootstrapper
 * ShowDriverDeprecationNotification
 * ShowBootstrapperDeprecationNotification
 * dashboard
 * addon-manager
 * default-storageclass
 * heapster
 * efk
 * ingress
 * insecure-registry
 * registry
 * registry-creds
 * freshpod
 * default-storageclass
 * storage-provisioner
 * storage-provisioner-gluster
 * metrics-server
 * nvidia-driver-installer
 * nvidia-gpu-device-plugin
 * logviewer
 * gvisor
 * hyperv-virtual-switch
 * disable-driver-mounts
 * cache
 * embed-certs
 * native-ssh

Available Commands:
  get         Gets the value of PROPERTY_NAME from the minikube config file
  set         Sets an individual value in a minikube config file
  unset       unsets an individual value in a minikube config file
  view        Display values currently set in the minikube config file
{{< /highlight  >}}



<h1>service</h1>


{{< highlight html >}}

Gets the kubernetes URL(s) for the specified service in your local cluster. In the case of multiple URLs they will be
printed one at a time.

Available Commands:
  list        Lists the URLs for the services in your local cluster

Options:
      --https=false: Open the service URL with https instead of http
      --interval=6: The initial time interval for each check that wait performs in seconds
  -n, --namespace='default': The service namespace
      --url=false: Display the kubernetes service URL in the CLI instead of opening it in the default browser
      --wait=20: Amount of time to wait for a service in seconds
{{< /highlight  >}}

<h1>mount</h1>

{{< highlight html >}}

Mounts the specified directory into minikube.

Options:
      --9p-version='9p2000.L': Specify the 9p version that the mount should use
      --gid='docker': Default group id used for the mount
      --ip='': Specify the ip that the mount should be setup on
      --kill=false: Kill the mount process spawned by minikube start
      --mode=493: File permissions used for the mount
      --msize=262144: The number of bytes to use for 9p packet payload
      --options=[]: Additional mount options, such as cache=fscache
      --type='9p': Specify the mount filesystem type (supported types: 9p)
{{< /highlight  >}}


<h1>mount</h1>



---------------------
Minikube parameters 
---------------------
* mapping of arguments to source code
* references
	- https://evalle.xyz/posts/configure-kube-apiserver-in-minikube/
	- https://www.marcolancini.it/2019/blog-deploy-kubernetes-lab/
