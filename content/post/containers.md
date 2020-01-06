---
date: "2020-01-05"
title: "containerd & runc"
author : "Nanik Tolaram (nanikjava@gmail.com)" 

---

<h1>containerd</h1>

* Following slides outline the role containerd plays including what kind of services it provides.
  
![ContaierDLayer](/media/containerd/containerd_a_role_in_container_ecosystem.png)

![WhatIsContainerD](/media/containerd/what_is_containerd.png)

* **containerd-shim** -- After runc runs the container, it exits (allowing us to not have any long-running processes responsible for our containers). The shim is the component which sits between containerd and runc to facilitate this. Containers does not died when dockerd orcontainerd died as it is 'attached' to the containerd-shim process. The containerd-shim process job is to monitor stdin(out) and report back the error code returned from exiting the container

* Some of containerd Makefile task:
    * *make bin/containerd-shim* -- building the containerd-shim app
* Following are some explanation about containerd source code:
    * *cmd/containerd*                -- contains the containerd daemon source code
    * *cmd/containerd-shim*           -- containerd-shim code
    * *cmd/containerd-shim-runc-v1*   -- containerd-shim-v1 code
    * *cmd/containerd-shim-runc-v2*   -- containerd-shim-v2 code
* Examples how to use containerd
    * Running ubuntu interactively
        * Make sure image is pulled using 'ctr image pull'
        * Run the following commands to run it interactively and kill
            * **sudo ./ctr run -t docker.io/library/ubuntu:latest u2** [ run the image ]
            * **sudo ./ctr container info ubuntulatest**
            {{< highlight text >}}
{
    "ID": "ubuntulatest",
    "Labels": {
        "io.containerd.image.config.stop-signal": "SIGTERM"
    },
    "Image": "docker.io/library/ubuntu:latest",
    "Runtime": {
        "Name": "io.containerd.runc.v2",
        "Options": {
            "type_url": "containerd.runc.v1.Options"
        }
    },
    "SnapshotKey": "ubuntulatest",
    "Snapshotter": "overlayfs",
    "CreatedAt": "2020-01-01T00:24:30.509643667Z",
    "UpdatedAt": "2020-01-01T00:24:30.509643667Z",
    "Extensions": null,
    "Spec": {
        "ociVersion": "1.0.1-dev",
        "process": {
            "user": {
                "uid": 0,
                "gid": 0
            },
            "args": [
                "/bin/bash"
            ],
            "env": [
                "PATH=/usr/local/sbin:/usr/local/bin:/usr/sbin:/usr/bin:/sbin:/bin"
            ],
            "cwd": "/",
            "capabilities": {
                "bounding": [
                    "CAP_CHOWN",
                    "CAP_DAC_OVERRIDE",
                    "CAP_FSETID",
                    "CAP_FOWNER",
                    "CAP_MKNOD",
                    "CAP_NET_RAW",
                    "CAP_SETGID",
                    "CAP_SETUID",
                    "CAP_SETFCAP",
                    "CAP_SETPCAP",
                    "CAP_NET_BIND_SERVICE",
                    "CAP_SYS_CHROOT",
                    "CAP_KILL",
                    "CAP_AUDIT_WRITE"
                ],
                "effective": [
                    "CAP_CHOWN",
                    "CAP_DAC_OVERRIDE",
                    "CAP_FSETID",
                    "CAP_FOWNER",
                    "CAP_MKNOD",
                    "CAP_NET_RAW",
                    "CAP_SETGID",
                    "CAP_SETUID",
                    "CAP_SETFCAP",
                    "CAP_SETPCAP",
                    "CAP_NET_BIND_SERVICE",
                    "CAP_SYS_CHROOT",
                    "CAP_KILL",
                    "CAP_AUDIT_WRITE"
                ],
                "inheritable": [
                    "CAP_CHOWN",
                    "CAP_DAC_OVERRIDE",
                    "CAP_FSETID",
                    "CAP_FOWNER",
                    "CAP_MKNOD",
                    "CAP_NET_RAW",
                    "CAP_SETGID",
                    "CAP_SETUID",
                    "CAP_SETFCAP",
                    "CAP_SETPCAP",
                    "CAP_NET_BIND_SERVICE",
                    "CAP_SYS_CHROOT",
                    "CAP_KILL",
                    "CAP_AUDIT_WRITE"
                ],
                "permitted": [
                    "CAP_CHOWN",
                    "CAP_DAC_OVERRIDE",
                    "CAP_FSETID",
                    "CAP_FOWNER",
                    "CAP_MKNOD",
                    "CAP_NET_RAW",
                    "CAP_SETGID",
                    "CAP_SETUID",
                    "CAP_SETFCAP",
                    "CAP_SETPCAP",
                    "CAP_NET_BIND_SERVICE",
                    "CAP_SYS_CHROOT",
                    "CAP_KILL",
                    "CAP_AUDIT_WRITE"
                ]
            },
            "rlimits": [
                {
                    "type": "RLIMIT_NOFILE",
                    "hard": 1024,
                    "soft": 1024
                }
            ],
            "noNewPrivileges": true
        },
        "root": {
            "path": "rootfs"
        },
        "mounts": [
            {
                "destination": "/proc",
                "type": "proc",
                "source": "proc",
                "options": [
                    "nosuid",
                    "noexec",
                    "nodev"
                ]
            },
            {
                "destination": "/dev",
                "type": "tmpfs",
                "source": "tmpfs",
                "options": [
                    "nosuid",
                    "strictatime",
                    "mode=755",
                    "size=65536k"
                ]
            },
            {
                "destination": "/dev/pts",
                "type": "devpts",
                "source": "devpts",
                "options": [
                    "nosuid",
                    "noexec",
                    "newinstance",
                    "ptmxmode=0666",
                    "mode=0620",
                    "gid=5"
                ]
            },
            {
                "destination": "/dev/shm",
                "type": "tmpfs",
                "source": "shm",
                "options": [
                    "nosuid",
                    "noexec",
                    "nodev",
                    "mode=1777",
                    "size=65536k"
                ]
            },
            {
                "destination": "/dev/mqueue",
                "type": "mqueue",
                "source": "mqueue",
                "options": [
                    "nosuid",
                    "noexec",
                    "nodev"
                ]
            },
            {
                "destination": "/sys",
                "type": "sysfs",
                "source": "sysfs",
                "options": [
                    "nosuid",
                    "noexec",
                    "nodev",
                    "ro"
                ]
            },
            {
                "destination": "/run",
                "type": "tmpfs",
                "source": "tmpfs",
                "options": [
                    "nosuid",
                    "strictatime",
                    "mode=755",
                    "size=65536k"
                ]
            }
        ],
        "linux": {
            "resources": {
                "devices": [
                    {
                        "allow": false,
                        "access": "rwm"
                    },
                    {
                        "allow": true,
                        "type": "c",
                        "major": 1,
                        "minor": 3,
                        "access": "rwm"
                    },
                    {
                        "allow": true,
                        "type": "c",
                        "major": 1,
                        "minor": 8,
                        "access": "rwm"
                    },
                    {
                        "allow": true,
                        "type": "c",
                        "major": 1,
                        "minor": 7,
                        "access": "rwm"
                    },
                    {
                        "allow": true,
                        "type": "c",
                        "major": 5,
                        "minor": 0,
                        "access": "rwm"
                    },
                    {
                        "allow": true,
                        "type": "c",
                        "major": 1,
                        "minor": 5,
                        "access": "rwm"
                    },
                    {
                        "allow": true,
                        "type": "c",
                        "major": 1,
                        "minor": 9,
                        "access": "rwm"
                    },
                    {
                        "allow": true,
                        "type": "c",
                        "major": 5,
                        "minor": 1,
                        "access": "rwm"
                    },
                    {
                        "allow": true,
                        "type": "c",
                        "major": 136,
                        "access": "rwm"
                    },
                    {
                        "allow": true,
                        "type": "c",
                        "major": 5,
                        "minor": 2,
                        "access": "rwm"
                    },
                    {
                        "allow": true,
                        "type": "c",
                        "major": 10,
                        "minor": 200,
                        "access": "rwm"
                    }
                ]
            },
            "cgroupsPath": "/default/ubuntulatest",
            "namespaces": [
                {
                    "type": "pid"
                },
                {
                    "type": "ipc"
                },
                {
                    "type": "uts"
                },
                {
                    "type": "mount"
                },
                {
                    "type": "network"
                }
            ],
            "maskedPaths": [
                "/proc/acpi",
                "/proc/asound",
                "/proc/kcore",
                "/proc/keys",
                "/proc/latency_stats",
                "/proc/timer_list",
                "/proc/timer_stats",
                "/proc/sched_debug",
                "/sys/firmware",
                "/proc/scsi"
            ],
            "readonlyPaths": [
                "/proc/bus",
                "/proc/fs",
                "/proc/irq",
                "/proc/sys",
                "/proc/sysrq-trigger"
            ]
        }
    }
}
            {{< /highlight >}}                        
            * **sudo ./ctr  t metrics ubuntulatest**
            {{< highlight text >}}
ID              TIMESTAMP                                  
ubuntulatest    2020-01-01 00:29:16.295322149 +0000 UTC    

METRIC                   VALUE                                                                                
memory.usage_in_bytes    1433600                                                                              
memory.limit_in_bytes    9223372036854771712                                                                  
memory.stat.cache        0                                                                                    
cpuacct.usage            19273664                                                                             
cpuacct.usage_percpu     [205817 2386548 4771841 148926 5529908 227097 2037362 0 2107441 0 1017213 841511]    
pids.current             1                                                                                    
pids.limit               0 
            {{< /highlight >}}                        
            * **sudo ./ctr  shim  --id ubuntulatest state**
            {{< highlight text >}}
{
    "id": "ubuntulatest",
    "bundle": "/run/containerd/io.containerd.runtime.v2.task/default/ubuntulatest",
    "pid": 16032,
    "status": 2,
    "stdin": "/run/containerd/fifo/249557939/ubuntulatest-stdin",
    "stdout": "/run/containerd/fifo/249557939/ubuntulatest-stdout",
    "stderr": "/run/containerd/fifo/249557939/ubuntulatest-stderr",
    "exited_at": "0001-01-01T00:00:00Z"
}
             {{< /highlight >}}
            * **sudo ./ctr t kill u2**                                 [ kill the container labelled u2 ]            
    * Another example to download hello world
        * **sudo ./ctr image pull docker.io/library/hello-world:latest**
{{< highlight text >}}
docker.io/library/hello-world:latest:                                             resolved       |++++++++++++++++++++++++++++++++++++++| 
index-sha256:4fe721ccc2e8dc7362278a29dc660d833570ec2682f4e4194f4ee23e415e1064:    done           |++++++++++++++++++++++++++++++++++++++| 
manifest-sha256:92c7f9c92844bbbb5d0a101b22f7c2a7949e40f8ea90c8b3bc396879d95e899a: done           |++++++++++++++++++++++++++++++++++++++| 
layer-sha256:1b930d010525941c1d56ec53b97bd057a67ae1865eebf042686d2a2d18271ced:    done           |++++++++++++++++++++++++++++++++++++++| 
config-sha256:fce289e99eb9bca977dae136fbe2a82b6b7d4c372474c9235adc1741675f587e:   done           |++++++++++++++++++++++++++++++++++++++| 
elapsed: 3.7 s                                                                    total:  4.8 Ki (1.3 KiB/s)                                       
unpacking linux/amd64 sha256:4fe721ccc2e8dc7362278a29dc660d833570ec2682f4e4194f4ee23e415e1064...
{{< /highlight >}}                        
        * **sudo ./ctr container create docker.io/library/hello-world:latest demo**
        * **sudo ./ctr task start demo**
* containerd utilise kernel feature called 'reaper' to reparent the container proces to the shim
  ![ContaierDLayer](/media/containerd/pr_set_child_reaper.png)
  Following shows the process structure when running a container using containerd
   {{< highlight text >}}
nanik     7741  3230  0  2019 ?        00:02:54  \_ /usr/libexec/gnome-terminal-server
nanik     7750  7741  0  2019 pts/1    00:00:00  |   \_ bash
.....
.....
nanik    19294  7741  0 13:45 pts/5    00:00:00  |   \_ bash
root      5123 19294  0 17:51 pts/5    00:00:00  |   |   \_ sudo ./ctr run -t docker.io/library/ubuntu:latest u13
root      5124  5123  0 17:51 pts/5    00:00:00  |   |       \_ ./ctr run -t docker.io/library/ubuntu:latest u13
nanik    18313  7741  0 16:11 pts/11   00:00:00  |   \_ bash
.....
.....
.....
.....
.....
root      5884  3230  0 17:52 ?        00:00:00  \_ /usr/bin/containerd-shim-runc-v2 -namespace default -id u13 -address /run/containerd/containerd.sock
root      5906  5884  0 17:52 ?        00:00:00      \_ /bin/bash
.....
.....
    {{< /highlight >}}
    As can be seen containerd uses shim called 'containerd-shim-run-v2'. Runc has been terminated after running the container and the shim takes over as the parent of the container. Containerd supports shim v2    
    ![ContaierDLayer](/media/containerd/shim_v2.png)
    
    The shim is executed out-of-process (executed with exec(..)) and the following are used to execute it:
   {{< highlight text >}}
 0 = {string} "-namespace"
 1 = {string} "default"
 2 = {string} "-address"
 3 = {string} "/run/containerd/containerd.sock"
 4 = {string} "-publish-binary"
 5 = {string} "/tmp/___containerd"
 6 = {string} "-id"
 7 = {string} "u8"
 8 = {string} "-debug"
 9 = {string} "start"
    {{< /highlight >}}

    The /tmp/__containerid contains the containerd executable.
    
    [Comment by Michael Crosby about shim](https://groups.google.com/forum/#!topic/docker-dev/zaZFlvIx1_k)
    
    {{< highlight text >}}
The shim allows for daemonless containers.  It basically sits as the parent of the container's process to facilitate a few things.

First it allows the runtimes, i.e. runc,to exit after it starts the container.  This way we don't have to have the long running runtime processes for containers.  When you start mysql you should only see the mysql process and the shim.

Second it keeps the STDIO and other fds open for the container incase containerd and/or docker both die.  If the shim was not running then the parent side of the pipes or the TTY master would be closed and the container would exit.  

Finally it allows the container's exit status to be reported back to a higher level tool like docker without having the be the actual parent of the container's process and do a wait.  
    {{< /highlight >}}

* containerd uses FIFO for reporting event and exit code and also for stdout and stdin
  {{< highlight text >}}
/run/containerd/fifo/195093460/<something_something>_stdout
/run/containerd/fifo/195093460/<something_something>_stdin
  {{< /highlight >}}

<h1>runc</h1>

* A lightweight binary that supports the OCI runtime-spec for running containers. Deals with the low-level interfacing with Linux capabilities like cgroups, namespaces, etc...
* runc looked for temp directory using "XDG_RUNTIME_DIR" eg:/run/user/1000/runc/
* runc have heavy dependencies on libcontainer.
* How 'runc' is used/executed inside containerd ?. Following are some explanation:
    * Containerd runs as a server where it receive GRPC command. The ctr CLI tool is the way to send command to run, stop, etc containers in containerd
    * When containerd receive command as such 
        **sudo ./ctr run  -t docker.io/library/ubuntu:latest  u67**, it will go through __services/tasks/service.go__ source code to prepare all the necessary data to spin off 'containerd-shim-runc-v2' executable.
    * Following is an example of the executable argument prepared when executing 'containerd-shim-runc-v2' 
      {{< highlight text >}}
 0 = {string} "-namespace"
 1 = {string} "default"
 2 = {string} "-address"
 3 = {string} "/run/containerd/containerd.sock"
 4 = {string} "-publish-binary"
 5 = {string} "/tmp/___containerd"
 6 = {string} "-id"
 7 = {string} "u8"
 8 = {string} "-debug"
 9 = {string} "start"
       {{< /highlight >}} 
      Full command used  -- **/usr/bin/containerd-shim-runc-v2 -namespace default -id u67 -address /run/containerd/containerd.sock**        
    * Following are the log output (debug log were added to trace soure code)  when 'containerd-shim-runc-v2' is running: 
      {{< highlight text >}}
time="2020-01-02T23:38:00.438682328+11:00" level=info msg=setupDumpStacks...
time="2020-01-02T23:38:00.438927931+11:00" level=info msg="calling newServer..."
time="2020-01-02T23:38:00.439033554+11:00" level=info msg="registering ttrpc server"
time="2020-01-02T23:38:00.439097725+11:00" level=info msg="calling serve..."
time="2020-01-02T23:38:00.439198636+11:00" level=info msg="calling handleSignals..."
time="2020-01-02T23:38:00.439220268+11:00" level=info msg="nanik starting signal loop" namespace=default path=/run/containerd/io.containerd.runtime.v2.task/default/u67 pid=17162
time="2020-01-02T23:38:00.439861913+11:00" level=info msg="Create is called inside  RegisterTaskService"
time="2020-01-02T23:38:00.439974576+11:00" level=info msg="container NANIK "
time="2020-01-02T23:38:00.440214591+11:00" level=info msg="--- CreateTaskRequest  &CreateTaskRequest{ID:u67,Bundle:/run/containerd/io.containerd.runtime.v2.task/default/u67,Rootfs:[&types.Mo
unt{Type:overlay,Source:overlay,Target:,Options:[workdir=/var/lib/containerd/io.containerd.snapshotter.v1.overlayfs/snapshots/133/work upperdir=/var/lib/containerd/io.containerd.snapshotter.
v1.overlayfs/snapshots/133/fs lowerdir=/var/lib/containerd/io.containerd.snapshotter.v1.overlayfs/snapshots/4/fs:/var/lib/containerd/io.containerd.snapshotter.v1.overlayfs/snapshots/3/fs:/va
r/lib/containerd/io.containerd.snapshotter.v1.overlayfs/snapshots/2/fs:/var/lib/containerd/io.containerd.snapshotter.v1.overlayfs/snapshots/1/fs],XXX_unrecognized:[],}],Terminal:true,Stdin:/
run/containerd/fifo/383837931/u67-stdin,Stdout:/run/containerd/fifo/383837931/u67-stdout,Stderr:,Checkpoint:,ParentCheckpoint:,Options:&types1.Any{TypeUrl:containerd.runc.v1.Options,Value:[]
,XXX_unrecognized:[],},XXX_unrecognized:[],}"
time="2020-01-02T23:38:00.440329033+11:00" level=info msg="--- rootfs  /run/containerd/io.containerd.runtime.v2.task/default/u67/rootfs"
time="2020-01-02T23:38:00.440350982+11:00" level=info msg="--- opts.BinaryName  "
time="2020-01-02T23:38:00.440367236+11:00" level=info msg="--- opts.Bundle  /run/containerd/io.containerd.runtime.v2.task/default/u67"
time="2020-01-02T23:38:00.441364775+11:00" level=info msg="--- calling p.create with context.Background.WithValue(type namespaces.namespaceKey, val default).WithValue(type metadata.mdOutgoin
gKey, val <not Stringer>).WithValue(type ttrpc.metadataKey, val <not Stringer>).WithValue(type shim.OptsKey, val <not Stringer>).WithValue(type log.loggerKey, val <not Stringer>).WithCancel.
WithCancel AND ... "
time="2020-01-02T23:38:00.441809346+11:00" level=info msg="------- inside init.gocontext.Background.WithValue(type namespaces.namespaceKey, val default).WithValue(type metadata.mdOutgoingKey
, val <not Stringer>).WithValue(type ttrpc.metadataKey, val <not Stringer>).WithValue(type shim.OptsKey, val <not Stringer>).WithValue(type log.loggerKey, val <not Stringer>).WithCancel.With
Cancel/run/containerd/io.containerd.runtime.v2.task/default/u67&{<nil> /run/containerd/io.containerd.runtime.v2.task/default/u67/init.pid 0xc000154520 false false false []}" runtime=io.conta
inerd.runc.v2
time="2020-01-02T23:38:00.503607036+11:00" level=info msg="Start is called inside  RegisterTaskService"
time="2020-01-02T23:38:00.503640676+11:00" level=info msg="v2/service Start"
ime="2020-01-02T17:33:35.454812846+11:00" level=info msg="v2/service Delete"
       {{< /highlight >}}

    * The final function that will execute 'runc' is inside __containerd/go-runc/runc.go__
      {{< highlight text >}}
func (r *Runc) Create(context context.Context, id, bundle string, opts *CreateOpts) error {}
      {{< /highlight >}}

    * Logging code was added inside the __Create(..)__ function and following is the output: 
      {{< highlight text >}}
--- args  [create --bundle /run/containerd/io.containerd.runtime.v2.task/default/u67]
--- cmd  /home/nanik/AndroidProjects/docker/docker/runc --root /run/containerd/runc/default --log /run/containerd/io.containerd.runtime.v2.task/default/u67/log.json --log-format json create
--bundle /run/containerd/io.containerd.runtime.v2.task/default/u67 --pid-file /run/containerd/io.containerd.runtime.v2.task/default/u67/init.pid --console-socket /tmp/pty415594316/pty.sock u
67

The command used to execute 'runc' is as follows
"/home/nanik/AndroidProjects/docker/docker/runc --root /run/containerd/runc/default --log /run/containerd/io.containerd.runtime.v2.task/default/u67/log.json --log-format json create --bundle /run/containerd/io.containerd.runtime.v2.task/default/u67 --pid-file /run/containerd/io.containerd.runtime.v2.task/default/u67/init.pid --console-socket /tmp/pty415594316/pty.sock u67"
     {{< /highlight >}}

* To use runc to see docker containers that are running
  {{< highlight text >}}
sudo ./runc --root /run/docker/runtime-runc/moby  list
 
ID                                                                 PID         STATUS      BUNDLE                                                                                                                 CREATED                          OWNER
f182f95645673b94af95495ea4c2a7c0f58dcce523f3d4e7174d7e482e136e08   12212       running     /run/containerd/io.containerd.runtime.v1.linux/moby/f182f95645673b94af95495ea4c2a7c0f58dcce523f3d4e7174d7e482e136e08   2020-01-05T21:28:05.371871841Z   root
  {{< /highlight >}}

    [Good explanation from here](https://stackoverflow.com/questions/57009928/runc-and-ctr-commands-do-not-show-docker-images-and-containers) 
  {{< highlight text >}}
The runtime (runc) uses so-called runtime root directory to store and obtain the information about containers. Under this root directory, runc places sub-directories (one per container), and each of them contains the state.json file, where the container state description resides.

The default location for runtime root directory is either /run/runc (for non-rootless containers) or $XDG_RUNTIME_DIR/runc (for rootless containers) - the latter also usually points to somewhere under /run (e.g. /run/user/$UID/runc).

When the container engine invokes runc, it may override the default runtime root directory and specify the custom one (--root option of runc). Docker uses this possibility, e.g. on my box, it specifies /run/docker/runtime-runc/moby as the runtime root.

That said, to make runc list see your Docker containers, you have to point it to Docker's runtime root directory by specifying --root option. Also, given that Docker containers are not rootless by default, you will need the appropriate privileges to access the runtime root (e.g. with sudo).

    So, that's how this should work:

    $ docker run -d alpine sleep 1000
    4acd4af5ba8da324b7a902618aeb3fd0b8fce39db5285546e1f80169f157fc69

    $ sudo runc --root /run/docker/runtime-runc/moby/ list
    ID                                                                 PID         STATUS      BUNDLE                                                                                                                               CREATED                          OWNER
    4acd4af5ba8da324b7a902618aeb3fd0b8fce39db5285546e1f80169f157fc69   18372       running     /run/docker/containerd/daemon/io.containerd.runtime.v1.linux/moby/4acd4af5ba8da324b7a902618aeb3fd0b8fce39db5285546e1f80169f157fc69   2019-07-12T17:33:23.401746168Z   root

As to images, you can not make runc see them, as it has no notion of image at all - instead, it operates on bundles. Creating the bundle (e.g. based on image) is responsibility of the caller (in your case - containerd).
  {{< /highlight >}}

<h1>libcontainer</h1>

* Library used inside runc for container operation
* [Explanation about libcontainer, runc and nsenter](https://stackoverflow.com/questions/42696589/libcontainer-runc-and-nsenter-bootstrap/42697174). More [in-depth explanation](https://groups.google.com/a/opencontainers.org/forum/#!msg/dev/CC1XH92oMrE/G1GRnBDGCAAJ)


<h1>Docker and lower level</h1>

* **Docker CLI (docker) - /usr/bin/docker**

    Docker is used as a reference to the whole set of docker tools and at the beginning it was a monolith. But now docker-cli is only responsible for user friendly communication with docker.

    So the command's like docker build ... docker run ... are handled by Docker CLI and result in the invocation of dockerd API.

* **Dockerd - /usr/bin/dockerd**

    The Docker daemon - dockerd listens for Docker API requests and manages host's Container life-cycles by utilizing contanerd

    dockerd can listen for Docker Engine API requests via three different types of Socket: unix, tcp, and fd. By default, a unix domain socket is created at /var/run/docker.sock, requiring either root permission, or docker group membership. On Systemd based systems, you can communicate with the daemon via Systemd socket activation, use dockerd -H fd://.

    There are many configuration options for the daemon, which are worth to check if you work with docker (dockerd).

    My impression is that dockerd is here to serve all the features of Docker (or Docker EE) platform, while actual container life-cycle management is "outsourced" to containerd.
    Containerd

* **containerd - /usr/bin/docker-containerd**

    containerd was introduced in Docker 1.11 and since then took main responsibilty of managing containers life-cycle. containerd is the executor for containers, but has a wider scope than just executing containers. So it also take care of:

      {{< highlight text >}}
Image push and pull
Managing of storage
Of course executing of Containers by calling runc with the right parameters to run containers...
Managing of network primitives for interfaces
Management of network namespaces containers to join existing namespaces
      {{< /highlight >}}

    containerd fully leverages the OCI runtime specification1, image format specifications and OCI reference implementation (runc). Because of its massive adoption, containerd is the industry standard for implementing OCI. It is currently available for Linux and Windows.

* **RunC - /usr/bin/docker-runc runc (OCI runtime) can be seen as component of containerd.**

    runc is a command line client for running applications packaged according to
    the OCI format and is a compliant implementation of the OCI spec.

    Containers are configured using bundles. A bundle for a container is a directory
    that includes a specification file named "config.json" and a root filesystem.
    The root filesystem contains the contents of the container.

    Assuming you have an OCI bundle you can execute the container 

* **containerd-ctr - /usr/bin/docker-containerd-ctr (docker-)containerd-ctr**

    it's barebone CLI (ctr) designed specifically for development and debugging purpose for direct communication with containerd. It's included in the releases of containerd. By that less interesting for docker users.

* **containerd-shim - /usr/bin/docker-containerd-shim**

    The shim allows for daemonless containers. According to Michael Crosby it's basically sits as the parent of the container's process to facilitate a few things.
    {{< highlight text >}}
First it allows the runtimes, i.e. runc,to exit after it starts the container. This way we don't have to have the long running runtime processes for containers.

Second it keeps the STDIO and other fds open for the container in case containerd and/or docker both die. If the shim was not running then the parent side of the pipes or the TTY master would be closed and the container would exit.

Finally it allows the container's exit status to be reported back to a higher level tool like docker without having the be the actual parent of the container's process and do a wait.
    {{< /highlight >}}

* Complete interaction between docker cli, dockerd, containerd, containerd-shim and runc
{{< highlight text >}}
dockerd is sent POST Containers Create
    ↳ dockerd finds the requested image
    ↳ A container object is created and stored for future use
    ↳ Directories on the file system are setup for use by the container
dockerd is sent a POST Containers Start
    ↳ An OCI spec is created for the container
    ↳ containerd is contacted to create the container
        ↳ containerd stores the container spec in a database
    ↳ containerd is contacted to start the container
        ↳  containerd creates a task for the container
            ↳  The task uses a shim to call runc create
        ↳  containerd starts the task
            ↳  The task uses the shim to call runc start
        ↳ The shim / containerd continue to monitor the container until completion
{{< /highlight >}}

<h1>Running container with runc</h1>

This following is step-by-step example on how to run OCI compliant image using runc. We going to use docker in this example.

* Checkout the **runc** project from https://github.com/opencontainers/runc and build it by running **make** 
* Download **exportrootfs.sh** from https://github.com/estesp/utils/blob/master/exportrootfs.sh and add it to your PATH. Make sure follow the instruction inside the script to compile uidmapshift.c and include that too in the PATH
* Make sure you have pull ubuntu:latest image using docker
* Run the image using the docker run command:
{{< highlight text >}}
docker run -it ubuntu:latest /bin/bash
{{< /highlight >}}    
* Get the container id using **docker ps**
{{< highlight text >}}
CONTAINER ID        IMAGE               COMMAND             CREATED             STATUS              PORTS               NAMES
ebfbbecaf715        ubuntu:latest       "/bin/bash"         38 minutes ago      Up 38 minutes                           zen_kirch
{{< /highlight >}}    
* Create another separate directory and cd into that directory. Run the following command
{{< highlight text >}}
sudo env "PATH=$PATH" exportrootfs.sh -u 0 -r 65536 ebf
{{< /highlight >}}
ebf is the container id shown in the output of **docker ps**
* You will have a **roootfs** directory in your current directory and it will look like the following
{{< highlight text >}}
rootfs
├── bin
├── boot
├── dev
├── etc
├── home
├── lib
├── lib64
├── media
├── mnt
├── opt
├── proc
├── root
├── run
├── sbin
├── srv
├── sys
├── tmp
├── usr
└── var
{{< /highlight >}}       
* Run the container using the following command

* Create the runtime spec using the command
{{< highlight text >}}
runc spec
{{< /highlight  >}}    
You will see a new file called **config.json**  
* Open config.json and modify the **args** to the following
{{< highlight text >}}    
"args": [
    "/bin/bash"
],
{{< /highlight  >}}            

* Execute the image using the following
{{< highlight text >}}    
sudo env "PATH=$PATH" runc run anycontainername
{{< /highlight  >}}
* You will see bash running
{{< highlight text >}}    
root@runc:/# cat /etc/lsb-release 
DISTRIB_ID=Ubuntu
DISTRIB_RELEASE=18.04
DISTRIB_CODENAME=bionic
DISTRIB_DESCRIPTION="Ubuntu 18.04.3 LTS"
root@runc:/# 
{{< /highlight  >}}

As can be seen the runc does not know how to pull, prepare, etc the image. It just knows that there is a root fileysystem with the config.json that it needs to run. The ubuntu container ran by the above example does not have network as this will be taken care by some other project and not by runc.

<h1>References</h1>

* Blogs
    * [Different parts of Docker explained](http://alexander.holbreich.org/docker-components-explained/)
    * [Docker container creation process](https://prefetch.net/blog/2018/02/19/how-the-docker-container-creation-process-works-from-docker-run-to-runc/)
    * [Michael Crosby Dockercon source example](https://github.com/crosbymichael/dockercon-2016)
    * [How docker works](https://cameronlonsdale.com/2019/03/25/how-does-docker-work/)
    * [Good presentation about containers](https://www.slideshare.net/PhilEstes)
    * [Containerd website](https://containerd.io/)
* Videos
    * [Traefik v2.0 in Docker + containerd Updates](https://www.youtube.com/watch?v=RP40Iv_0yvA)
    * [Deep Dive into firecracker-containerd](https://www.youtube.com/watch?v=0wEiizErKZw)
    * [Containerd: The Universal Container Runtime - Justin Cormack, Docker](https://www.youtube.com/watch?v=cfhnioURGdE)
    * [Looking Under The Hood: containerD](https://www.youtube.com/watch?v=fIRaPGxhsH0)
    * [runc - how it works video](https://www.youtube.com/watch?v=ZAhzoz2zJj8)
* Forums    
    * [Answers to some interesting low-level questions](https://stackoverflow.com/users/7191047/danila-kiver)
* Source Code
    * [runc source Code](https://github.com/opencontainers/runc)
    * [Containerd source code](https://github.com/containerd/containerd)
