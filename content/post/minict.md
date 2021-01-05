---
date: "2021-01-04"
title: "Dissecting minict"
author : "Nanik Tolaram (nanikjava@gmail.com)" 

---

minict is a small interesting project that shows how containers works, here is the [link to the github](https://github.com/Ripolak/minict).

You can think of the project as Docker v0.0.1 that provide very simple functionality to play around with containers. 

Here are some of the functionality it provides out-of-the box

```
COMMANDS:
   pull             Pull an image from Dockerhub or a different container registry.
   run              Run a new container.
   start            Start an existing container that was exited.
   rm               Remove an existing container.
   list-containers  List all current containers.
   list-images      List all images.
   help, h          Shows a list of commands or help for one command
```

This article will tear down the project to look at how it works internally and to see how it is able to provide the container functionality using other open source projects.


## pull

This command is to pull container image from _*docker.io*_ registry and store it locally. This command should be executed first before any other command.

{{< highlight bash >}}
minict pull --image ubuntu:20.04 
{{< /highlight >}}

Following is the log output (to enable more verbose logging look under the Hacking section)

```
DEBU[0000] reference rewritten from 'docker.io/library/ubuntu:20.04' to 'docker.io/library/ubuntu:20.04' 
DEBU[0000] Trying to pull "docker.io/library/ubuntu:20.04" 
DEBU[0000] Credentials not found                        
DEBU[0000] Using registries.d directory /etc/containers/registries.d for sigstore configuration 
DEBU[0000]  No signature storage configuration found for docker.io/library/ubuntu:20.04 
DEBU[0000] Looking for TLS certificates and private keys in /etc/docker/certs.d/docker.io 
DEBU[0000] GET https://registry-1.docker.io/v2/         
DEBU[0001] Ping https://registry-1.docker.io/v2/ status 401 
DEBU[0001] GET https://auth.docker.io/token?scope=repository%3Alibrary%2Fubuntu%3Apull&service=registry.docker.io 
DEBU[0004] GET https://registry-1.docker.io/v2/library/ubuntu/manifests/20.04 
DEBU[0005] Using blob info cache at /var/lib/containers/cache/blob-info-cache-v1.boltdb 
DEBU[0005] Source is a manifest list; copying (only) instance sha256:4e4bc990609ed865e07afc8427c30ffdddca5153fd4e82c20d8f0783a291e241 
DEBU[0005] GET https://registry-1.docker.io/v2/library/ubuntu/manifests/sha256:4e4bc990609ed865e07afc8427c30ffdddca5153fd4e82c20d8f0783a291e241 
DEBU[0006] IsRunningImageAllowed for image docker:docker.io/library/ubuntu:20.04 
DEBU[0006]  Using default policy section                
DEBU[0006]  Requirement 0: allowed                      
DEBU[0006] Overall: allowed                             
Getting image source signatures
DEBU[0006] Manifest has MIME type application/vnd.docker.distribution.manifest.v2+json, ordered candidate list [application/vnd.oci.image.manifest.v1+json] 
DEBU[0006] Downloading /v2/library/ubuntu/blobs/sha256:da7391352a9bb76b292a568c066aa4c3cbae8d494e6a3c68e3c596d34f7c75f8 
DEBU[0006] GET https://registry-1.docker.io/v2/library/ubuntu/blobs/sha256:da7391352a9bb76b292a568c066aa4c3cbae8d494e6a3c68e3c596d34f7c75f8 
DEBU[0009] Detected compression format gzip             
DEBU[0009] Using original blob without modification     
Copying blob da7391352a9b [======================================] 27.2MiB / 27.2MiB
DEBU[0043] Downloading /v2/library/ubuntu/blobs/sha256:14428a6d4bcdba49a64127900a0691fb00a3f329aced25eb77e3b65646638f8d 
Copying blob da7391352a9b done
DEBU[0044] Detected compression format gzip             
DEBU[0044] Using original blob without modification     
DEBU[0044] Downloading /v2/library/ubuntu/blobs/sha256:2c2d948710f21ad82dce71743b1654b45acb5c059cf5c19da491582cef6f2601 
Copying blob da7391352a9b done
Copying blob 14428a6d4bcd done
Copying blob da7391352a9b done
Copying blob 14428a6d4bcd done
Copying blob 2c2d948710f2 done
DEBU[0046] Downloading /v2/library/ubuntu/blobs/sha256:f643c72bc25212974c16f3348b3a898b1ec1eb13ec1539e10a103e6e217eb2f1 
DEBU[0046] GET https://registry-1.docker.io/v2/library/ubuntu/blobs/sha256:f643c72bc25212974c16f3348b3a898b1ec1eb13ec1539e10a103e6e217eb2f1 
DEBU[0047] No compression detected                      
DEBU[0047] Using original blob without modification     
Copying config aa23411143 done
Writing manifest to image destination
Storing signatures
2021/01/05 20:05:27  info Image pulled successfully.

Process finished with exit code 0
```

The image is downloaded inside **/var/lib/minict/images** and it looks like the following

{{< highlight text >}}
/var/lib/minict/
├── containers
└── images
    └── ubuntu
        ├── blobs
        │   └── sha256
        │       ├── 14428a6d4bcdba49a64127900a0691fb00a3f329aced25eb77e3b65646638f8d
        │       ├── 2c2d948710f21ad82dce71743b1654b45acb5c059cf5c19da491582cef6f2601
        │       ├── 5d52e1388dedc5da07eebd41b0b1c189183c537be51b9a9d6d5e82385373b2f6
        │       ├── aa23411143b1e053a8458b3ea4252fb14570a0621ba0e3d26f6143616a874db1
        │       └── da7391352a9bb76b292a568c066aa4c3cbae8d494e6a3c68e3c596d34f7c75f8
        ├── index.json
        └── oci-layout
{{< /highlight >}}

Following outlined the different files downloaded from the pull process.

***`index.json`***

This file contains information about the image (eg: platform)


{{< highlight json >}}
    schemaVersion: 2
    manifests: [
      {
        "mediaType": "application/vnd.oci.image.manifest.v1+json",
        "digest": "sha256:5d52e1388dedc5da07eebd41b0b1c189183c537be51b9a9d6d5e82385373b2f6",
        "size": 658,
        "annotations": {
          "org.opencontainers.image.ref.name": "20.04"
        },
        "platform": {
          "architecture": "amd64",
          "os": "linux"
        }
      }
    ]
{{< /highlight >}}


***`oci-layout`***

This file contains information about the oci-layout information used to store the image

{{< highlight json >}}
    imageLayoutVersion: "1.0.0"
{{< /highlight >}}


***`blobs/sha256`***

***5d52e1388dedc5da07eebd41b0b1c189183c537be51b9a9d6d5e82385373b2f6***

This file contains information about the files that has been downloaded including size, type and shasum


{{< highlight json >}}
    schemaVersion: 2
    config: {
      "mediaType": "application/vnd.oci.image.config.v1+json",
      "digest": "sha256:aa23411143b1e053a8458b3ea4252fb14570a0621ba0e3d26f6143616a874db1",
      "size": 2427
    }
    layers: [
      {
        "mediaType": "application/vnd.oci.image.layer.v1.tar+gzip",
        "digest": "sha256:da7391352a9bb76b292a568c066aa4c3cbae8d494e6a3c68e3c596d34f7c75f8",
        "size": 28563271
      },
      {
        "mediaType": "application/vnd.oci.image.layer.v1.tar+gzip",
        "digest": "sha256:14428a6d4bcdba49a64127900a0691fb00a3f329aced25eb77e3b65646638f8d",
        "size": 847
      },
      {
        "mediaType": "application/vnd.oci.image.layer.v1.tar+gzip",
        "digest": "sha256:2c2d948710f21ad82dce71743b1654b45acb5c059cf5c19da491582cef6f2601",
        "size": 162
      }
    ]
{{< /highlight >}}

***aa23411143b1e053a8458b3ea4252fb14570a0621ba0e3d26f6143616a874db1***

This file contains information about the container, for example: it contains information about the command that will be executed when the container startsup, in our example it will be ***/bin/bash***

{{< highlight json >}}
    created: "2020-11-25T22:25:29.546718343Z"
    architecture: "amd64"
    os: "linux"
    config: {
      "Env": [
        "PATH=/usr/local/sbin:/usr/local/bin:/usr/sbin:/usr/bin:/sbin:/bin"
      ],
      "Cmd": [
        "/bin/bash"
      ]
    }
    rootfs: {
      "type": "layers",
      "diff_ids": [
        "sha256:bacd3af13903e13a43fe87b6944acd1ff21024132aad6e74b4452d984fb1a99a",
        "sha256:9069f84dbbe96d4c50a656a05bbe6b6892722b0d1116a8f7fd9d274f4e991bf6",
        "sha256:f6253634dc78da2f2e3bee9c8063593f880dc35d701307f30f65553e0f50c18c"
      ]
    }
    history: [
      {
        "created": "2020-11-25T22:25:26.245907708Z",
        "created_by": "/bin/sh -c #(nop) ADD file:4f15c4475fbafb3fe335e415e3ea1ac416c34af911fcdfe273c5759438aa8eb4 in / "
      },
      {
        "created": "2020-11-25T22:25:27.346756278Z",
        "created_by": "/bin/sh -c set -xe \t\t&& echo '#!/bin/sh' > /usr/sbin/policy-rc.d \t&& echo 'exit 101' >> /usr/sbin/policy-rc.d \t&& chmod +x /usr/sbin/policy-rc.d \t\t&& dpkg-divert --local --rename --add /sbin/initctl \t&& cp -a /usr/sbin/policy-rc.d /sbin/initctl \t&& sed -i 's/^exit.*/exit 0/' /sbin/initctl \t\t&& echo 'force-unsafe-io' > /etc/dpkg/dpkg.cfg.d/docker-apt-speedup \t\t&& echo 'DPkg::Post-Invoke { \"rm -f /var/cache/apt/archives/*.deb /var/cache/apt/archives/partial/*.deb /var/cache/apt/*.bin || true\"; };' > /etc/apt/apt.conf.d/docker-clean \t&& echo 'APT::Update::Post-Invoke { \"rm -f /var/cache/apt/archives/*.deb /var/cache/apt/archives/partial/*.deb /var/cache/apt/*.bin || true\"; };' >> /etc/apt/apt.conf.d/docker-clean \t&& echo 'Dir::Cache::pkgcache \"\"; Dir::Cache::srcpkgcache \"\";' >> /etc/apt/apt.conf.d/docker-clean \t\t&& echo 'Acquire::Languages \"none\";' > /etc/apt/apt.conf.d/docker-no-languages \t\t&& echo 'Acquire::GzipIndexes \"true\"; Acquire::CompressionTypes::Order:: \"gz\";' > /etc/apt/apt.conf.d/docker-gzip-indexes \t\t&& echo 'Apt::AutoRemove::SuggestsImportant \"false\";' > /etc/apt/apt.conf.d/docker-autoremove-suggests"
      },
      {
        "created": "2020-11-25T22:25:28.342445422Z",
        "created_by": "/bin/sh -c [ -z \"$(apt-get indextargets)\" ]",
        "empty_layer": true
      },
      {
        "created": "2020-11-25T22:25:29.343142847Z",
        "created_by": "/bin/sh -c mkdir -p /run/systemd && echo 'docker' > /run/systemd/container"
      },
      {
        "created": "2020-11-25T22:25:29.546718343Z",
        "created_by": "/bin/sh -c #(nop)  CMD [\"/bin/bash\"]",
        "empty_layer": true
      }
    ]
{{< /highlight >}}

The code uses library from  [**github.com/containers/image**](https://github.com/containers/image) to download and store the image from a remote registry. This library is quite extensive and provide a lot of functionality to work with OCI based images, in order it's functionality there is another tool that uses this library extensively called [skopeo](https://github.com/containers/skopeo)

## run

{{< highlight bash >}}
minict run --image ubuntu:20.04 --name ubuntu-nanik 
{{< /highlight >}}

This command is to run the downloaded images as container. In our example we are running the ubuntu:20.04 image that we have previously downloaded, the container will be given label ubuntu-nanik.

Following is the output showing minict is reading the image files and trying to start it up.

```
2021/01/05 20:00:38  info Container process exited. It can be started again using the 'start' option.
2021/01/05 20:00:38  info unpacking bundle ...     
2021/01/05 20:00:38  info unpack rootfs: /var/lib/minict/containers/ubuntu-nanik/rootfs
2021/01/05 20:00:38  info unpack layer: sha256:da7391352a9bb76b292a568c066aa4c3cbae8d494e6a3c68e3c596d34f7c75f8
2021/01/05 20:00:39  info unpack layer: sha256:14428a6d4bcdba49a64127900a0691fb00a3f329aced25eb77e3b65646638f8d
2021/01/05 20:00:39  info unpack layer: sha256:2c2d948710f21ad82dce71743b1654b45acb5c059cf5c19da491582cef6f2601
2021/01/05 20:00:39  info ... done                 
2021/01/05 20:00:39  info computing filesystem manifest ...
2021/01/05 20:00:40  info ... done                 
2021/01/05 20:00:40  info unpacked image bundle: /var/lib/minict/containers/ubuntu-nanik
2021/01/05 20:00:40 Failed to mount tmpfs to /dev due to invalid argument
2021/01/05 20:00:40 Failed to mount devpts to /dev/pts due to no such file or directory
2021/01/05 20:00:40 Failed to mount shm to /dev/shm due to no such file or directory
2021/01/05 20:00:40 Failed to mount mqueue to /dev/mqueue due to no such file or directory
```

Once the container is up and running you will be given bash command to work with.

The run command is using the [umoci](https://github.com/opencontainers/umoci) project to work with containers. The way the umoci library works to start an image as container is as follows

* Perform validation for:
	- check to make sure the the oci-layout file is valid
	- the blobs directory exist
	- index.json file exist
* Do unpacking when using the Unpack(..) function as follows:
	- read the index.json
	- convert the layer information (from 5d52e1388dedc5da07eebd41b0b1c189183c537be51b9a9d6d5e82385373b2f6 file) to internal struct
	- create all the necessary directories inside /var/lib/minict/containers/ubuntu-nanik to host the filesystem (including rootfs)
	- unpack filesystem from all the image files into /rootfs
	- create config.json containing the following information
{{< highlight text >}}
{
	"ociVersion": "1.0.0",
	"process": {
	        "terminal": true,
	        "user": {
	                "uid": 0,
	                "gid": 0
	        },
	        "args": [
	                "/bin/bash"
	        ],
	        "env": [
	                "PATH=/usr/local/sbin:/usr/local/bin:/usr/sbin:/usr/bin:/sbin:/bin",
	                "TERM=xterm",
	                "HOME=/root"
	        ],
	        "cwd": "/",
	        "capabilities": {
	                "bounding": [
	                        "CAP_AUDIT_WRITE",
	                        "CAP_KILL",
	                        "CAP_NET_BIND_SERVICE"
	                ],
	                "effective": [
	                        "CAP_AUDIT_WRITE",
	                        "CAP_KILL",
	                        "CAP_NET_BIND_SERVICE"
	                ],
	                "inheritable": [
	                        "CAP_AUDIT_WRITE",
	                        "CAP_KILL",
	                        "CAP_NET_BIND_SERVICE"
	                ],
	                "permitted": [
	                        "CAP_AUDIT_WRITE",
	                        "CAP_KILL",
	                        "CAP_NET_BIND_SERVICE"
	                ],
	                "ambient": [
	                        "CAP_AUDIT_WRITE",
	                        "CAP_KILL",
	                        "CAP_NET_BIND_SERVICE"
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
	"hostname": "umoci-default",
	"mounts": [
	        {
	                "destination": "/proc",
	                "type": "proc",
	                "source": "proc"
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
	                        "mode=0620"
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
	                "type": "bind",
	                "source": "/sys",
	                "options": [
	                        "rbind",
	                        "nosuid",
	                        "noexec",
	                        "nodev",
	                        "ro"
	                ]
	        },
	        {
	                "destination": "/etc/resolv.conf",
	                "type": "bind",
	                "source": "/etc/resolv.conf",
	                "options": [
	                        "noexec",
	                        "nosuid",
	                        "rbind",
	                        "ro"
	                ]
	        }
	],
	"annotations": {
	        "org.opencontainers.image.architecture": "amd64",
	        "org.opencontainers.image.author": "",
	        "org.opencontainers.image.created": "2020-11-25T22:25:29.546718343Z",
	        "org.opencontainers.image.exposedPorts": "",
	        "org.opencontainers.image.os": "linux",
	        "org.opencontainers.image.stopSignal": ""
	},
	"linux": {
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
	                        "type": "user"
	                }
	        ],
	        "maskedPaths": [
	                "/proc/kcore",
	                "/proc/latency_stats",
	                "/proc/timer_list",
	                "/proc/timer_stats",
	                "/proc/sched_debug",
	                "/sys/firmware",
	                "/proc/scsi"
	        ],
	        "readonlyPaths": [
	                "/proc/asound",
	                "/proc/bus",
	                "/proc/fs",
	                "/proc/irq",
	                "/proc/sys",
	                "/proc/sysrq-trigger"
	        ]
	}
}
{{< /highlight >}}
	- read config.json to construct internal config object
	- construct the command that will be executed to run the container and apply namespaces

## Hacking
* Add the following code inside oci/pull.go under function func PullImage(..) to enable the download reporting
{{< highlight go >}}
return copy.Image(ctx, policyContext, destRef, srcRef, &copy.Options{ ReportWriter: os.Stdout})
{{< /highlight >}}

* Add the following inside main.go to increase the logging level
{{< highlight go >}}
logrus.SetLevel(8)
{{< /highlight >}}




