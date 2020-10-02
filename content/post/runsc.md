---
date: "2020-10-02"
title: "gvisor"
author : "Nanik Tolaram (nanikjava@gmail.com)" 
---

This article will explain about the container runtime called [gvisor](http://www.gvisor.dev). The article focus more on how Sentry provides extra security layer to gvisor and how it intercepts system calls and interact with the application running inside container.

## Building gvisor 

* Make sure your docker is working properly as the build system required docker container to build the binaries.

* Build gvisor issuing the command
{{< highlight bash >}}
make runsc
{{< /highlight >}}
the binary is located inside  ./bazel-gvisor/bazel-out/k8-opt/bin/runsc/linux_amd64_pure_stripped copy the runsc binary to
{{< highlight bash >}}
/usr/local/bin/
{{< /highlight >}}
* Copy the following to a file called daemon.json inside /etc/docker
{{< highlight json >}}
{
	"default-runtime": "runsc",
	"runtimes": {
		"runc-local": {
			"path": "runc"
		},
		"runsc": {
			"path": "/usr/local/bin/runsc",
			"runtimeArgs": [
				"--debug-log=/tmp/runsc/",
				"--debug",
				"--strace"
			]
		}
	}
}
{{< /highlight >}}

* Run the following docker command to test runsc
{{< highlight bash >}}
docker run --runtime=runsc -it ubuntu  dmesg  
{{< /highlight >}}

  The result will be as follows
{{< highlight text >}}
[    0.000000] Starting gVisor...
[    0.380255] Letting the watchdogs out...
[    0.671842] Waiting for children...
[    1.111248] Moving files to filing cabinet...
[    1.537250] Checking naughty and nice process list...
[    1.632947] Creating process schedule...
[    2.025866] Mounting deweydecimalfs...
[    2.054713] Reading process obituaries...
[    2.239299] Daemonizing children...
[    2.485680] Reticulating splines...
[    2.558058] Consulting tar man page...
[    2.741916] Ready!
{{< /highlight >}}

* Run the following to test docker using the normal runc
{{< highlight bash >}}
docker run --privileged  --runtime=runc-local -it ubuntu  dmesg
{{< /highlight >}}

  The result is as follows
{{< highlight text >}}
[    0.000000] microcode: microcode updated early to revision 0x2f, date = 2019-11-12
[    0.000000] Linux version 5.4.0-7642-generic (buildd@lcy01-amd64-007) (gcc version 9.3.0 (Ubuntu 9.3.0-10ubuntu2)) #46~1598628707~20.04~040157c-Ubuntu SMP Fri Aug 28 18:02:16 UTC  (Ubuntu 5.4.0-7642.46~1598628707~20.04~040157c-generic 5.4.44)
[    0.000000] Command line: BOOT_IMAGE=/vmlinuz-5.4.0-7642-generic root=/dev/mapper/data-root ro quiet splash vt.handoff=7
[    0.000000] KERNEL supported cpus:
[    0.000000]   Intel GenuineIntel
[    0.000000]   AMD AuthenticAMD
[    0.000000]   Hygon HygonGenuine
[    0.000000]   Centaur CentaurHauls
...
...
...
...
{{< /highlight >}}

## Sentry the 'Secret Sauce'

Noticed the difference in output that you get running runsc compared to runc ?. The dmesg output from runsc shows that the kernel that it is running is NOT the host kernel, unlike what we see in the output when using runc. This is the 'special sauce' about gvisor.

gvisor provides a 'usermode' Linux Kernel or known as UML. Inside runsc there is a sub-module called Sentry which act as thin layer of kernel, this does not mean that it's job is to replace Kernel. Sentry provide an extra layer of 'safety' that intercepts system calls from application that is running inside a container. This prevents the application from having any direct interaction with the host through syscalls. 

gVisor supports two methods of redirecting syscalls 
	* ptrace-mode - uses ptrace in the Linux kernel to forward syscalls to the sentry and 
	* KVM-mode - uses KVM to trap syscalls before they hit the Linux kernel so they can be forwarded to the sentry.

![alt text](https://gvisor.dev/docs/Layers.png)

Read the [gvisor docs](https://gvisor.dev/docs/) to get more intro about what is it all about and what kind of security it provides 


## Internals

We will discuss more in details at how gvisor really works internally. This is based on my personal experiment and research, so there could be some information that is not correct.

Following are some of the topics and concepts that you need to understand to really understand how Sentry works and the security functionality it provides for gvisor.

* [vDSO](https://en.wikipedia.org/wiki/VDSO)
* [Seccomp](https://en.wikipedia.org/wiki/Seccomp)	
* [ptrace](https://en.wikipedia.org/wiki/Ptrace)
* [ELF Auxiliary Vectors](https://refspecs.linuxfoundation.org/LSB_1.3.0/IA64/spec/auxiliaryvector.html)
* [syscalls](https://man7.org/linux/man-pages/man2/syscalls.2.html)

To get a better understand how the whole container stack works (docker, containerd, etc) refer to my [other posting](https://nanikgolang.netlify.app/post/containers/)

### Execution

When container is launched using runsc it does what is shown in the diagram below

![alt text](/media/runsc/execution.jpg)


What the diagram shows there are few things that is happening behind the scenes before your container is executed. The first command that is triggered by docker to run runsc is as follows 

```
/usr/local/bin/runsc --debug-log=/tmp/runsc/ --debug --strace --root /var/run/docker/runtime-runc/moby --log /run/containerd/io.containerd.runtime.v2.task/moby/e488ee2bef4cfa13e82dc38c7e7db759ad3a8f89e02218a0339adb79a47193af/log.json --log-format json create --bundle /run/containerd/io.containerd.runtime.v2.task/moby/e488ee2bef4cfa13e82dc38c7e7db759ad3a8f89e02218a0339adb79a47193af --pid-file /run/containerd/io.containerd.runtime.v2.task/moby/e488ee2bef4cfa13e82dc38c7e7db759ad3a8f89e02218a0339adb79a47193af/init.pid --console-socket /tmp/pty993587804/pty.sock e488ee2bef4cfa13e82dc38c7e7db759ad3a8f89e02218a0339adb79a47193af
```

Following on executing the above command runsc will executed itself few times as follows sequence:

(1) execute gofer for setting up Capabilities for directories

```
/proc/self/exe --root=/var/run/docker/runtime-runc/moby --debug=true --log=/run/containerd/io.containerd.runtime.v2.task/moby/e488ee2bef4cfa13e82dc38c7e7db759ad3a8f89e02218a0339adb79a47193af/log.json --log-format=json --debug-log=/tmp/runsc/ --strace=true --log-fd=3 --debug-log-fd=4 gofer --bundle /run/containerd/io.containerd.runtime.v2.task/moby/e488ee2bef4cfa13e82dc38c7e7db759ad3a8f89e02218a0339adb79a47193af --spec-fd=5 --mounts-fd=6 --io-fds=7 --io-fds=8 --io-fds=9 --io-fds=10
```

(2) execute gofer to run as normal

```
/proc/self/exe --root=/var/run/docker/runtime-runc/moby --debug=true --log=/run/containerd/io.containerd.runtime.v2.task/moby/e488ee2bef4cfa13e82dc38c7e7db759ad3a8f89e02218a0339adb79a47193af/log.json --log-format=json --debug-log=/tmp/runsc/ --strace=true --log-fd=3 --debug-log-fd=4 gofer --bundle /run/containerd/io.containerd.runtime.v2.task/moby/e488ee2bef4cfa13e82dc38c7e7db759ad3a8f89e02218a0339adb79a47193af --spec-fd=5 --mounts-fd=6 --io-fds=7 --io-fds=8 --io-fds=9 --io-fds=10 --apply-caps=false --setup-root=false
```

(3) execute sandbox to setup root
	
```
/proc/self/exe --root=/var/run/docker/runtime-runc/moby --debug=true --log=/run/containerd/io.containerd.runtime.v2.task/moby/e488ee2bef4cfa13e82dc38c7e7db759ad3a8f89e02218a0339adb79a47193af/log.json --log-format=json --debug-log=/tmp/runsc/ --strace=true --log-fd=3 --debug-log-fd=4 boot --bundle=/run/containerd/io.containerd.runtime.v2.task/moby/e488ee2bef4cfa13e82dc38c7e7db759ad3a8f89e02218a0339adb79a47193af --controller-fd=5 --mounts-fd=6 --spec-fd=7 --start-sync-fd=8 --io-fds=9 --io-fds=10 --io-fds=11 --io-fds=12 --stdio-fds=13 --stdio-fds=14 --stdio-fds=15 --setup-root --cpu-num 4 e488ee2bef4cfa13e82dc38c7e7db759ad3a8f89e02218a0339adb79a47193af]
```

(4) execute sandbox to setup everything else (Sentry, ptrace, etc)

```
/proc/self/exe --root=/var/run/docker/runtime-runc/moby --debug=true --log=/run/containerd/io.containerd.runtime.v2.task/moby/e488ee2bef4cfa13e82dc38c7e7db759ad3a8f89e02218a0339adb79a47193af/log.json --log-format=json --debug-log=/tmp/runsc/ --strace=true --log-fd=3 --debug-log-fd=4 boot --bundle=/run/containerd/io.containerd.runtime.v2.task/moby/e488ee2bef4cfa13e82dc38c7e7db759ad3a8f89e02218a0339adb79a47193af --controller-fd=5 --mounts-fd=6 --spec-fd=7 --start-sync-fd=8 --io-fds=9 --io-fds=10 --io-fds=11 --io-fds=12 --stdio-fds=13 --stdio-fds=14 --stdio-fds=15 --cpu-num 4 e488ee2bef4cfa13e82dc38c7e7db759ad3a8f89e02218a0339adb79a47193af
```

(5) start container to run the sandbox along with the container and it's application

```
/usr/local/bin/runsc --debug-log=/tmp/runsc/ --debug --strace --root /var/run/docker/runtime-runc/moby --log /run/containerd/io.containerd.runtime.v2.task/moby/e488ee2bef4cfa13e82dc38c7e7db759ad3a8f89e02218a0339adb79a47193af/log.json --log-format json start e488ee2bef4cfa13e82dc38c7e7db759ad3a8f89e02218a0339adb79a47193af
```

The following diagram shows the different process that was created during the lifecycle of runsc. The number in the diagram inside bracket is the PID of the runsc process when it was running.

![alt text](/media/runsc/differentthreadsexecution.jpg)

### vDSO

vdso stands for virtual ELF dynamic shared object. It is a small shared library that the kernel automatically maps into the address space of ALL user-space applications. The reason why we have vdso is to make syscall faster, but due to it's small space only 4 different functions are provided
{{< highlight text >}}
clock_gettime
getcpu
gettimeofday
time
{{< /highlight >}}

Sentry implement it's own VDSO which is located inside vdso directory in the source code. 

When application is loaded into memory to be executed the ELF loader populate the vdso address using the auxiliary vectors. These vectors are the mechanism to transfer some OS specific information to the program interpreter (e.g. ld) and the process. This task is normally done by the elf loader in Linux but in gvisor it is done by sentry/loader package. These vectors are put on the process stack along with other information like argc, argv, envp. After stack initialization, normal stack will look something like this: 

{{< highlight text >}}
position            content                     size (bytes) + comment
------------------------------------------------------------------------
stack pointer ->  [ argc = number of args ]     4
            [ argv[0] (pointer) ]         4   (program name)
            [ argv[1] (pointer) ]         4
            [ argv[..] (pointer) ]        4 * x
            [ argv[n - 1] (pointer) ]     4
            [ argv[n] (pointer) ]         4   (= NULL)

            [ envp[0] (pointer) ]         4
            [ envp[1] (pointer) ]         4
            [ envp[..] (pointer) ]        4
            [ envp[term] (pointer) ]      4   (= NULL)

            [ auxv[0] (Elf32_auxv_t) ]    8
            [ auxv[1] (Elf32_auxv_t) ]    8
            [ auxv[..] (Elf32_auxv_t) ]   8
            [ auxv[term] (Elf32_auxv_t) ] 8   (= AT_NULL vector)

            [ padding ]                   0 - 16

            [ argument ASCIIZ strings ]   >= 0
            [ environment ASCIIZ str. ]   >= 0

(0xbffffffc)      [ end marker ]                4   (= NULL)

(0xc0000000)      < bottom of stack >           0   (virtual)
{{< /highlight >}}
In Sentry all the heavy lifting of the vDSO, etc are done inside the loader.go during the time when application is being loaded into memory as shown below.


{{< highlight go >}}
[pkg/sentry/loader/loader.go]


func Load(ctx context.Context, args LoadArgs, extraAuxv []arch.AuxEntry, vdso *VDSO) (abi.OS, arch.Context, string, *syserr.Error) {
	// Load the executable itself.
	loaded, ac, file, newArgv, err := loadExecutable(ctx, args)
	...
	...
	// Add generic auxv entries.
	auxv := append(loaded.auxv, arch.Auxv{
		arch.AuxEntry{linux.AT_UID, usermem.Addr(c.RealKUID.In(c.UserNamespace).OrOverflow())},
		arch.AuxEntry{linux.AT_EUID, usermem.Addr(c.EffectiveKUID.In(c.UserNamespace).OrOverflow())},
		arch.AuxEntry{linux.AT_GID, usermem.Addr(c.RealKGID.In(c.UserNamespace).OrOverflow())},
		arch.AuxEntry{linux.AT_EGID, usermem.Addr(c.EffectiveKGID.In(c.UserNamespace).OrOverflow())},
		// The conditions that require AT_SECURE = 1 never arise. See
		// kernel.Task.updateCredsForExecLocked.
		arch.AuxEntry{linux.AT_SECURE, 0},
		arch.AuxEntry{linux.AT_CLKTCK, linux.CLOCKS_PER_SEC},
		arch.AuxEntry{linux.AT_EXECFN, execfn},
		arch.AuxEntry{linux.AT_RANDOM, random},
		arch.AuxEntry{linux.AT_PAGESZ, usermem.PageSize},
		arch.AuxEntry{linux.AT_SYSINFO_EHDR, vdsoAddr},
	}...)
	auxv = append(auxv, extraAuxv...)

	sl, err := stack.Load(newArgv, args.Envv, auxv)
	...
	...
{{< /highlight >}}

specifically this particular line of code which sets the vDSO address

{{< highlight go >}}
arch.AuxEntry{linux.AT_SYSINFO_EHDR, vdsoAddr},
{{< /highlight >}}

### Seccomp

{{< highlight text>}}
seccomp (short for secure computing mode) is a computer security facility in the Linux kernel. seccomp allows a process to make a one-way transition into a "secure" state where it cannot make any system calls except exit(), sigreturn(), read() and write() to already-open file descriptors. Should it attempt any other system calls, the kernel will terminate the process with SIGKILL or SIGSYS.[1][2] In this sense, it does not virtualize the system's resources but isolates the process from them entirely. 
{{< /highlight >}}

gvisor is known as 'seccomp on steroids' because it uses seccomp to setup security filters. Which means that any application running inside a container is not calling the host kernel, but instead it will be calling gvisor kernel. 

Following is an example of the seccomp filters that are installed by gvisor (obtained from the log)

{{< highlight text>}}
I0929 04:00:54.926295       1 seccomp.go:61] Installing seccomp filters for 58 syscalls (action=kill process)
D0929 04:00:54.926349       1 seccomp.go:171] syscall filter read: [] => 0x616c6c6f77
D0929 04:00:54.926371       1 seccomp.go:171] syscall filter write: [] => 0x616c6c6f77
D0929 04:00:54.926388       1 seccomp.go:171] syscall filter close: [] => 0x616c6c6f77
D0929 04:00:54.926398       1 seccomp.go:171] syscall filter fstat: [] => 0x616c6c6f77
D0929 04:00:54.926408       1 seccomp.go:171] syscall filter lseek: [] => 0x616c6c6f77
D0929 04:00:54.926582       1 seccomp.go:171] syscall filter mmap: [( * * * == 0x1 ) ( * * * == 0x22 ) ( * * * == 0x32 )] => 0x616c6c6f77
D0929 04:00:54.926612       1 seccomp.go:171] syscall filter mprotect: [] => 0x616c6c6f77
D0929 04:00:54.926619       1 seccomp.go:171] syscall filter munmap: [] => 0x616c6c6f77
D0929 04:00:54.926624       1 seccomp.go:171] syscall filter rt_sigprocmask: [] => 0x616c6c6f77
D0929 04:00:54.926630       1 seccomp.go:171] syscall filter rt_sigreturn: [] => 0x616c6c6f77
D0929 04:00:54.926636       1 seccomp.go:171] syscall filter pread64: [] => 0x616c6c6f77
D0929 04:00:54.926641       1 seccomp.go:171] syscall filter pwrite64: [] => 0x616c6c6f77
D0929 04:00:54.926647       1 seccomp.go:171] syscall filter sched_yield: [] => 0x616c6c6f77
D0929 04:00:54.926652       1 seccomp.go:171] syscall filter madvise: [] => 0x616c6c6f77
D0929 04:00:54.926658       1 seccomp.go:171] syscall filter dup: [] => 0x616c6c6f77
D0929 04:00:54.926666       1 seccomp.go:171] syscall filter nanosleep: [] => 0x616c6c6f77
D0929 04:00:54.926675       1 seccomp.go:171] syscall filter getpid: [] => 0x616c6c6f77
D0929 04:00:54.926681       1 seccomp.go:171] syscall filter accept: [] => 0x616c6c6f77
D0929 04:00:54.926687       1 seccomp.go:171] syscall filter sendmsg: [( * * == 0x0 ) ( * * == 0x4040 )] => 0x616c6c6f77
D0929 04:00:54.926697       1 seccomp.go:171] syscall filter recvmsg: [( * * == 0x60 ) ( * * == 0x62 )] => 0x616c6c6f77
D0929 04:00:54.926710       1 seccomp.go:171] syscall filter shutdown: [( * == 0x2 )] => 0x616c6c6f77
D0929 04:00:54.926717       1 seccomp.go:171] syscall filter socketpair: [( == 0x1 == 0x80005 == 0x0 )] => 0x616c6c6f77
D0929 04:00:54.926725       1 seccomp.go:171] syscall filter clone: [( == 0xd0f00 * == 0x0 == 0x0 * ) ( == 0x50f00 * == 0x0 == 0x0 * )] => 0x616c6c6f77
D0929 04:00:54.926741       1 seccomp.go:171] syscall filter exit: [] => 0x616c6c6f77
D0929 04:00:54.926754       1 seccomp.go:171] syscall filter fcntl: [( * == 0x3 ) ( * == 0x4 ) ( * == 0x1 ) ( * == 0x409 )] => 0x616c6c6f77
D0929 04:00:54.926768       1 seccomp.go:171] syscall filter fsync: [] => 0x616c6c6f77
D0929 04:00:54.926773       1 seccomp.go:171] syscall filter ftruncate: [] => 0x616c6c6f77
D0929 04:00:54.926779       1 seccomp.go:171] syscall filter fchmod: [] => 0x616c6c6f77
D0929 04:00:54.926785       1 seccomp.go:171] syscall filter gettimeofday: [] => 0x616c6c6f77
D0929 04:00:54.926790       1 seccomp.go:171] syscall filter sigaltstack: [] => 0x616c6c6f77
D0929 04:00:54.926801       1 seccomp.go:171] syscall filter fstatfs: [] => 0x616c6c6f77
D0929 04:00:54.926808       1 seccomp.go:171] syscall filter mlock: [( * == 0x1000 )] => 0x616c6c6f77
D0929 04:00:54.926815       1 seccomp.go:171] syscall filter arch_prctl: [( == 0x1002 )] => 0x616c6c6f77
D0929 04:00:54.926828       1 seccomp.go:171] syscall filter gettid: [] => 0x616c6c6f77
D0929 04:00:54.926842       1 seccomp.go:171] syscall filter futex: [( * == 0x80 * * == 0x0 ) ( * == 0x81 * * == 0x0 ) ( * == 0x0 * * ) ( * == 0x1 * * )] => 0x616c6c6f77
D0929 04:00:54.926859       1 seccomp.go:171] syscall filter getdents64: [] => 0x616c6c6f77
D0929 04:00:54.926866       1 seccomp.go:171] syscall filter restart_syscall: [] => 0x616c6c6f77
D0929 04:00:54.926872       1 seccomp.go:171] syscall filter clock_gettime: [] => 0x616c6c6f77
D0929 04:00:54.926877       1 seccomp.go:171] syscall filter exit_group: [] => 0x616c6c6f77
D0929 04:00:54.926886       1 seccomp.go:171] syscall filter epoll_ctl: [] => 0x616c6c6f77
D0929 04:00:54.926892       1 seccomp.go:171] syscall filter tgkill: [( == 0x1 )] => 0x616c6c6f77
D0929 04:00:54.926904       1 seccomp.go:171] syscall filter openat: [] => 0x616c6c6f77
D0929 04:00:54.926910       1 seccomp.go:171] syscall filter mkdirat: [] => 0x616c6c6f77
D0929 04:00:54.926916       1 seccomp.go:171] syscall filter mknodat: [] => 0x616c6c6f77
D0929 04:00:54.926921       1 seccomp.go:171] syscall filter fchownat: [] => 0x616c6c6f77
D0929 04:00:54.926927       1 seccomp.go:171] syscall filter newfstatat: [] => 0x616c6c6f77
D0929 04:00:54.926932       1 seccomp.go:171] syscall filter unlinkat: [] => 0x616c6c6f77
D0929 04:00:54.926938       1 seccomp.go:171] syscall filter renameat: [] => 0x616c6c6f77
D0929 04:00:54.926943       1 seccomp.go:171] syscall filter linkat: [] => 0x616c6c6f77
D0929 04:00:54.926949       1 seccomp.go:171] syscall filter symlinkat: [] => 0x616c6c6f77
D0929 04:00:54.926954       1 seccomp.go:171] syscall filter readlinkat: [] => 0x616c6c6f77
D0929 04:00:54.926960       1 seccomp.go:171] syscall filter ppoll: [] => 0x616c6c6f77
D0929 04:00:54.926965       1 seccomp.go:171] syscall filter utimensat: [] => 0x616c6c6f77
D0929 04:00:54.926973       1 seccomp.go:171] syscall filter epoll_pwait: [( * * * * == 0x0 )] => 0x616c6c6f77
D0929 04:00:54.926987       1 seccomp.go:171] syscall filter fallocate: [( * == 0x0 )] => 0x616c6c6f77
D0929 04:00:54.926995       1 seccomp.go:171] syscall filter eventfd2: [( == 0x0 == 0x0 )] => 0x616c6c6f77
D0929 04:00:54.927008       1 seccomp.go:171] syscall filter getrandom: [] => 0x616c6c6f77
D0929 04:00:54.927019       1 seccomp.go:171] syscall filter memfd_create: [] => 0x616c6c6f77
{{< /highlight >}}

### Syscalls

![gvisorkernelsyscall](/media/runsc/gvisorkernelsyscall.png)

As can be seen from the above diagram when the container app is doing syscall it will be using the syscalls that is inside Sentry and not the host kernel.

When running a container using runsc run 

{{< highlight bash>}}
cat /proc/self/maps
{{< /highlight >}}

We will see the following memory maps


{{< highlight text>}}
562059612000-562059614000 r--p 00000000 00:0e 19                         /usr/bin/cat
562059614000-562059619000 r-xp 00002000 00:0e 19                         /usr/bin/cat
562059619000-56205961c000 r--p 00007000 00:0e 19                         /usr/bin/cat
56205961c000-56205961d000 r--p 00009000 00:0e 19                         /usr/bin/cat
56205961d000-56205961e000 rw-p 0000a000 00:0e 19                         /usr/bin/cat
56205961e000-56205963f000 rw-p 00000000 00:00 0                          [heap]
7f74f24eb000-7f74f2ceb000 rw-p 00000000 00:00 0                          [stack]
7f986152b000-7f986154d000 rw-p 00000000 00:00 0 
7f986154d000-7f9861572000 r--p 00000000 00:0e 30                         /usr/lib/x86_64-linux-gnu/libc-2.31.so
7f9861572000-7f98616ea000 r-xp 00025000 00:0e 30                         /usr/lib/x86_64-linux-gnu/libc-2.31.so
7f98616ea000-7f9861734000 r--p 0019d000 00:0e 30                         /usr/lib/x86_64-linux-gnu/libc-2.31.so
7f9861734000-7f9861735000 ---p 001e7000 00:0e 30                         /usr/lib/x86_64-linux-gnu/libc-2.31.so
7f9861735000-7f9861738000 r--p 001e7000 00:0e 30                         /usr/lib/x86_64-linux-gnu/libc-2.31.so
7f9861738000-7f986173b000 rw-p 001ea000 00:0e 30                         /usr/lib/x86_64-linux-gnu/libc-2.31.so
7f986173b000-7f9861741000 rw-p 00000000 00:00 0 
7f9861743000-7f9861744000 r--p 00000000 00:00 0                          [vvar]
7f9861744000-7f9861746000 r-xp 00000000 00:00 0 
7f9861746000-7f9861747000 r--p 00000000 00:0e 27                         /usr/lib/x86_64-linux-gnu/ld-2.31.so
7f9861747000-7f986176a000 r-xp 00001000 00:0e 27                         /usr/lib/x86_64-linux-gnu/ld-2.31.so
7f986176a000-7f9861772000 r--p 00024000 00:0e 27                         /usr/lib/x86_64-linux-gnu/ld-2.31.so
7f9861773000-7f9861774000 r--p 0002c000 00:0e 27                         /usr/lib/x86_64-linux-gnu/ld-2.31.so
7f9861774000-7f9861775000 rw-p 0002d000 00:0e 27                         /usr/lib/x86_64-linux-gnu/ld-2.31.so
7f9861775000-7f9861776000 rw-p 00000000 00:00 0 
ffffffffff600000-ffffffffff601000 r-xp 00000000 00:00 0                  [vsyscall]
{{< /highlight >}}

in comparison following is from local Linux 

{{< highlight text>}}
55b8227e4000-55b8227e6000 r--p 00000000 fd:01 11272329                   /usr/bin/cat
55b8227e6000-55b8227eb000 r-xp 00002000 fd:01 11272329                   /usr/bin/cat
55b8227eb000-55b8227ee000 r--p 00007000 fd:01 11272329                   /usr/bin/cat
55b8227ee000-55b8227ef000 r--p 00009000 fd:01 11272329                   /usr/bin/cat
55b8227ef000-55b8227f0000 rw-p 0000a000 fd:01 11272329                   /usr/bin/cat
55b823606000-55b823627000 rw-p 00000000 00:00 0                          [heap]
7f2e3747a000-7f2e3749c000 rw-p 00000000 00:00 0 
7f2e3749c000-7f2e38396000 r--p 00000000 fd:01 11409222                   /usr/lib/locale/locale-archive
7f2e38396000-7f2e383bb000 r--p 00000000 fd:01 11808327                   /usr/lib/x86_64-linux-gnu/libc-2.31.so
7f2e383bb000-7f2e38533000 r-xp 00025000 fd:01 11808327                   /usr/lib/x86_64-linux-gnu/libc-2.31.so
7f2e38533000-7f2e3857d000 r--p 0019d000 fd:01 11808327                   /usr/lib/x86_64-linux-gnu/libc-2.31.so
7f2e3857d000-7f2e3857e000 ---p 001e7000 fd:01 11808327                   /usr/lib/x86_64-linux-gnu/libc-2.31.so
7f2e3857e000-7f2e38581000 r--p 001e7000 fd:01 11808327                   /usr/lib/x86_64-linux-gnu/libc-2.31.so
7f2e38581000-7f2e38584000 rw-p 001ea000 fd:01 11808327                   /usr/lib/x86_64-linux-gnu/libc-2.31.so
7f2e38584000-7f2e3858a000 rw-p 00000000 00:00 0 
7f2e385a0000-7f2e385a1000 r--p 00000000 fd:01 11800376                   /usr/lib/x86_64-linux-gnu/ld-2.31.so
7f2e385a1000-7f2e385c4000 r-xp 00001000 fd:01 11800376                   /usr/lib/x86_64-linux-gnu/ld-2.31.so
7f2e385c4000-7f2e385cc000 r--p 00024000 fd:01 11800376                   /usr/lib/x86_64-linux-gnu/ld-2.31.so
7f2e385cd000-7f2e385ce000 r--p 0002c000 fd:01 11800376                   /usr/lib/x86_64-linux-gnu/ld-2.31.so
7f2e385ce000-7f2e385cf000 rw-p 0002d000 fd:01 11800376                   /usr/lib/x86_64-linux-gnu/ld-2.31.so
7f2e385cf000-7f2e385d0000 rw-p 00000000 00:00 0 
7fffbe390000-7fffbe3b1000 rw-p 00000000 00:00 0                          [stack]
7fffbe3bd000-7fffbe3c0000 r--p 00000000 00:00 0                          [vvar]
7fffbe3c0000-7fffbe3c1000 r-xp 00000000 00:00 0                          [vdso]
ffffffffff600000-ffffffffff601000 --xp 00000000 00:00 0                  [vsyscall]
{{< /highlight >}}

### Loading, Executing and Switching app/kernel

Since Sentry works like a Linux kernel it is responsible in reading, loading and executing the executable that the container is going to run. This means it provides functionality like how an [ELF loader](https://en.wikipedia.org/wiki/Executable_and_Linkable_Format) does in Linux. The kernel code can be found inside sentry/kernel folder

![loadImageAndExecute](/media/runsc/loadImageAndExecute.jpg)

As shown in the diagram below when the application is doing a system call (eg: writing to a file, console, etc) it will raise a signal that will be intercepted by Sentry. Once the system call is done byt Sentry it goes back to the application, and the cycle continue.

![switchingbetweenkernelapp](/media/runsc/switchingbetweenkernelapp.jpg)

This is possible with the help of ptrace.


### ptrace

When gvisor starts up it create what is called 'master' thread and it creates different thread by cloning this 'master' thread and it enable ptracing on the thread to enable it to capture system calls when the application is run.

Basically the application is 'wrapped' inside a goroutine that is part of gvisor and this way gvisor (including Sentry) has capability to control the application.

### Log

Following is log that shows the main functionality of running the app and it's interaction with Sentry. The log message is not the full logs generated by gvisor, only the one that is interesting (which has my name attached to it) is shown here. 

{{< highlight text>}}
I1002 20:55:52.477178   57407 container.go:328] Nanik - container.New
I1002 20:55:52.477202   57407 container.go:868] Nanik - createGoferProcess
I1002 10:55:52.517996   57434 loader.go:491] Nanik - createPlatform
I1002 10:55:52.518030   57434 ptrace.go:207] Nanik - ptrace.New
I1002 10:55:52.518359   57434 subprocess_linux.go:135] Nanik - beforeFork
I1002 10:55:52.518557   57434 subprocess_linux.go:150] Nanik - afterFork pid & tgid 57454
I1002 10:55:52.519704   57434 subprocess_amd64.go:238] Nanik - Attempt an emulation.
I1002 10:55:52.519733   57434 subprocess_amd64.go:243] Nanik -  t.wait
I1002 10:55:52.519747   57434 subprocess_amd64.go:262] Nanik -  NOT syscallEvent | syscall.SIGTRAP
I1002 10:55:52.519752   57434 subprocess_amd64.go:238] Nanik - Attempt an emulation.
I1002 10:55:52.519759   57434 subprocess_amd64.go:243] Nanik -  t.wait
I1002 10:55:52.519769   57434 subprocess_amd64.go:246] Nanik -  syscallEvent | syscall.SIGTRAP
I1002 10:55:52.519921   57434 subprocess_linux.go:55] Nanik - createStub
I1002 10:55:52.520130   57434 subprocess_linux.go:135] Nanik - beforeFork
I1002 10:55:52.520263   57434 subprocess_linux.go:150] Nanik - afterFork pid & tgid 57455
I1002 10:55:52.520605   57434 ptrace_unsafe.go:125] Nanik - clone..cloning 57455 -- 57455
I1002 10:55:52.520694   57434 subprocess.go:178] Nanik - requests to create threads...cloning the firstThread 57455
I1002 10:55:52.752987   57434 loader.go:581] Nanik - run  l.createContainerProcess(true, l.sandboxID, &l.root, ep);
I1002 10:55:52.753274   57434 loader.go:734] Nanik - setupContainerFS
I1002 10:55:52.784717   57434 path.go:37] Nanik - ResolveExecutablePath args &{Filename: File:<nil> Argv:[/hello] Envv:[PATH=/usr/local/sbin:/usr/local/bin:/usr/sbin:/usr/bin:/sbin:/bin HOSTNAME=a8e612468f0b] WorkingDirectory:/ Credentials:0xc0002b85a0 FDTable:   fd:0 => name host:[1]
I1002 10:55:52.784827   57434 path.go:48] Nanik - ResolveExecutablePath path.IsAbs(name)
I1002 10:55:52.785174   57434 loader.go:755] Nanik - createContainerProcess l.k.CreateProcess(info.procArgs) {Filename:/hello File:<nil> Argv:[/hello] Envv:[PATH=/usr/local/sbin:/usr/local/bin:/usr/sbin:/usr/bin:/sbin:/bin HOSTNAME=a8e612468f0b HOME=/] WorkingDirectory:/ Credentials:0xc0002b85a0 FDTable:       fd:0 => name host:[1]
I1002 10:55:52.788418   57434 loader.go:180] Nanik - loadExecutable elfMagic
I1002 10:55:52.788437   57434 elf.go:643] Nanik - loadELF for /hello 
I1002 10:55:52.789010   57434 task_start.go:243] Nanik - assignTIDsLocked 1
I1002 10:55:52.789870   57434 kernel.go:1068] Nanik - Kernel Start()
I1002 10:55:52.789899   57434 task_start.go:307] Nanik - task_start Start 1
I1002 10:55:52.789959   57434 task_run.go:70] Nanik - run task_run.go 1
I1002 10:55:52.790006   57434 task_usermem.go:35] Nanik - task_usermem Activate
I1002 10:55:52.790172   57434 ptrace_unsafe.go:125] Nanik - clone..cloning 57455 -- 57455
I1002 10:55:52.793677   57434 subprocess.go:178] Nanik - requests to create threads...cloning the firstThread 57455
I1002 10:55:52.793942   57434 subprocess_linux.go:254] Nanik - createStub 57515
I1002 10:55:52.794118   57434 ptrace_unsafe.go:125] Nanik - clone..cloning 57515 -- 57515
I1002 10:55:52.794232   57434 subprocess.go:178] Nanik - requests to create threads...cloning the firstThread 57515
I1002 10:55:52.794447   57434 task_run.go:81] Nanik - run task_run.go t.interruptSelf()  1
I1002 10:55:52.794482   57434 task_run.go:97] Nanik -  execute
I1002 10:55:52.794508   57434 task_run.go:97] Nanik -  execute
I1002 10:55:52.794529   57434 task_run.go:97] Nanik -  execute
I1002 10:55:52.794540   57434 ptrace.go:103] Nanik - going to go inside switchToApp
I1002 10:55:52.794549   57434 subprocess.go:495] Nanik -switchToApp
I1002 10:55:52.794598   57434 ptrace_unsafe.go:125] Nanik - clone..cloning 57515 -- 57515
I1002 10:55:52.794691   57434 subprocess.go:178] Nanik - requests to create threads...cloning the firstThread 57515
I1002 10:55:52.794789   57434 subprocess.go:547] Nanik - NOT isSingleStepping 57518
I1002 10:55:52.794947   57434 subprocess.go:558] Nanik - after t.wait
I1002 10:55:52.794966   57434 subprocess.go:561] Nanik - Refresh all registers
I1002 10:55:52.794980   57434 subprocess.go:589] Nanik - outside Is it a system call?
I1002 10:55:52.795038   57434 task_run.go:97] Nanik -  execute
I1002 10:55:52.795051   57434 ptrace.go:103] Nanik - going to go inside switchToApp
I1002 10:55:52.795061   57434 subprocess.go:495] Nanik -switchToApp
I1002 10:55:52.795075   57434 subprocess.go:547] Nanik - NOT isSingleStepping 57518
I1002 10:55:52.795123   57434 subprocess.go:558] Nanik - after t.wait
I1002 10:55:52.795138   57434 subprocess.go:561] Nanik - Refresh all registers
I1002 10:55:52.795152   57434 subprocess.go:589] Nanik - outside Is it a system call?
I1002 10:55:52.795194   57434 task_run.go:97] Nanik -  execute
I1002 10:55:52.795206   57434 ptrace.go:103] Nanik - going to go inside switchToApp
I1002 10:55:52.795215   57434 subprocess.go:495] Nanik -switchToApp
I1002 10:55:52.795232   57434 subprocess.go:547] Nanik - NOT isSingleStepping 57518
I1002 10:55:52.795273   57434 subprocess.go:558] Nanik - after t.wait
I1002 10:55:52.795285   57434 subprocess.go:561] Nanik - Refresh all registers
I1002 10:55:52.795298   57434 subprocess.go:589] Nanik - outside Is it a system call?
I1002 10:55:52.795338   57434 task_run.go:97] Nanik -  execute
I1002 10:55:52.795350   57434 ptrace.go:103] Nanik - going to go inside switchToApp
I1002 10:55:52.795359   57434 subprocess.go:495] Nanik -switchToApp
I1002 10:55:52.795372   57434 subprocess.go:547] Nanik - NOT isSingleStepping 57518
I1002 10:55:52.795403   57434 subprocess.go:558] Nanik - after t.wait
I1002 10:55:52.795414   57434 subprocess.go:561] Nanik - Refresh all registers
I1002 10:55:52.795427   57434 subprocess.go:589] Nanik - outside Is it a system call?
I1002 10:55:52.795466   57434 task_run.go:97] Nanik -  execute
I1002 10:55:52.795478   57434 ptrace.go:103] Nanik - going to go inside switchToApp
I1002 10:55:52.795494   57434 subprocess.go:495] Nanik -switchToApp
I1002 10:55:52.795508   57434 subprocess.go:547] Nanik - NOT isSingleStepping 57518
I1002 10:55:52.795547   57434 subprocess.go:558] Nanik - after t.wait
I1002 10:55:52.795561   57434 subprocess.go:561] Nanik - Refresh all registers
I1002 10:55:52.795575   57434 subprocess.go:579] Nanik - a system call
I1002 10:55:52.795592   57434 task_syscall.go:200] Nanik -  doSyscallEnter 158
I1002 10:55:52.795685   57434 task_run.go:97] Nanik -  execute
I1002 10:55:52.795722   57434 ptrace.go:103] Nanik - going to go inside switchToApp
I1002 10:55:52.795733   57434 subprocess.go:495] Nanik -switchToApp
I1002 10:55:52.795748   57434 subprocess.go:547] Nanik - NOT isSingleStepping 57518
I1002 10:55:52.795781   57434 subprocess.go:558] Nanik - after t.wait
I1002 10:55:52.795793   57434 subprocess.go:561] Nanik - Refresh all registers
I1002 10:55:52.795807   57434 subprocess.go:579] Nanik - a system call
I1002 10:55:52.795820   57434 task_syscall.go:200] Nanik -  doSyscallEnter 218
I1002 10:55:52.795863   57434 task_run.go:97] Nanik -  execute
I1002 10:55:52.795872   57434 ptrace.go:103] Nanik - going to go inside switchToApp
I1002 10:55:52.795880   57434 subprocess.go:495] Nanik -switchToApp
I1002 10:55:52.795894   57434 subprocess.go:547] Nanik - NOT isSingleStepping 57518
..
..
..
..
..
I1002 10:55:52.796112   57434 task_run.go:97] Nanik -  execute
I1002 10:55:52.796130   57434 ptrace.go:103] Nanik - going to go inside switchToApp
I1002 10:55:52.796147   57434 subprocess.go:495] Nanik -switchToApp
I1002 10:55:52.796170   57434 subprocess.go:547] Nanik - NOT isSingleStepping 57518
I1002 10:55:52.796219   57434 subprocess.go:558] Nanik - after t.wait
I1002 10:55:52.796247   57434 subprocess.go:561] Nanik - Refresh all registers
I1002 10:55:52.796283   57434 subprocess.go:579] Nanik - a system call
I1002 10:55:52.796305   57434 task_syscall.go:200] Nanik -  doSyscallEnter 231
I1002 10:55:52.796642   57434 task_run.go:97] Nanik -  execute
I1002 10:55:52.796683   57434 task_run.go:97] Nanik -  execute
..
..
..

{{< /highlight >}}

The command ran to generate the above log is as follows

{{< highlight bash>}}
docker run --runtime=runsc hello-world 
{{< /highlight >}}

### Other Notes

gettimeofday() on Linux is what's called a vsyscall and/or vdso. 

{{< highlight bash>}}
0x00000034f408c2d4 : mov    $0xffffffffff600000,%rax
0x00000034f408c2db : callq  *%rax
{{< /highlight >}}

The address 0xffffffffff600000 is the vsyscall page (on x86_64). The mechanism maps a specific kernel-created code page into user memory, so that a few "syscalls" can be made without the overhead of a user/kernel context switch, but rather as "ordinary" function call.

Syscalls generally create a lot of overhead, and given the abundance of gettimeofday() calls, one would prefer not to use a syscall. To that end, Linux kernel may map one or two special areas into each program, called vdso and vsyscall. Your implementation of gettimeofday() seems to be using vsyscall:

{{< highlight bash>}}
mov    $0xffffffffff600000,%rax
{{< /highlight >}}

This is the standard address of vsyscall map. Try cat /proc/self/maps to see that mapping. The idea behind vsyscall is that kernel provides fast user-space implementations of some functions, and libc just calls them.

Read this nice article for more details.

* https://stackoverflow.com/questions/7266813/how-does-the-gettimeofday-syscall-work
* https://lwn.net/Articles/604287/ and https://lwn.net/Articles/604515/

### Resources

#### __gvisor__

* https://research.cs.wisc.edu/multifacet/papers/vee20_blending.pdf
* https://blog.loof.fr/2018/06/gvisor-in-depth.html
* [User Mode Linux is a good book](https://www.oreilly.com/library/view/user-mode-linux/0131865056/) and gvisor application kernel works the same like that.
* https://wenboshen.org/posts/2018-12-01-sectainer.html
{{< highlight text>}}
This is straight forward, sentry process acts as tracer while application process is the tracee. Application process system call will stop by PTRACE and wait for sentry to handle. Sentry can emulate the system call, replace the real system with getpid(), and return.
{{< /highlight >}}
* https://archive.fosdem.org/2020/schedule/event/containers_k8s_runtimes/attachments/slides/3751/export/events/attachments/containers_k8s_runtimes/slides/3751/below_kubernetes.pdf 
* [Understanding about Ring and Sentry](https://groups.google.com/g/gvisor-users/c/15FfcCilupo/m/9ARSLnH3BQAJ)
* [Inside gvisor](https://wenboshen.org/posts/2018-12-25-gvisor-inside.html#sentry)

#### __Seccomp__

* https://blog.selectel.com/containers-security-seccomp/
* https://eigenstate.org/notes/seccomp

#### __ptrace__
* [Excellent sample from Liz Rice](https://github.com/FuzzyLogic/gotrace/tree/master)
* [Another good example](https://github.com/lunixbochs/ghostrace)

#### __user mode Linux & Linux in general__
* https://christine.website/blog/howto-usermode-linux-2019-07-07
* https://github.com/firecracker-microvm/firecracker/blob/master/docs/rootfs-and-kernel-setup.md
* https://david942j.blogspot.com/2018/10/note-learning-kvm-implement-your-own.html
* [This explains what the different rings means in OS context](https://stackoverflow.com/questions/18717016/what-are-ring-0-and-ring-3-in-the-context-of-operating-systems)
	- Linux uses Ring 0 (Kernel mode) and 3 (user mode)
* [Article on how to run program without an operating system](https://stackoverflow.com/questions/22054578/how-to-run-a-program-without-an-operating-system/32483545#32483545)
* [Very good resource to learn bare metal OS](https://github.com/cirosantilli/x86-bare-metal-examples)
* [Searchable Linux Syscall Table for x86 and x86_64](https://filippo.io/linux-syscall-table/)
* [ELF internal for dynamic and static](https://www.gabriel.urdhr.fr/2015/01/22/elf-linking/)
