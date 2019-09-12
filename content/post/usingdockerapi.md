---
date: "2019-09-01"
title: "Playing with Docker SDK (Work in progress)"
author : "Nanik Tolaram (nanikjava@gmail.com)" 

---
This article will walk through on how to use the docker SDK using Go. Docker provide a very rich SDK allowing developers to communicate with the docker daemon.

<h1>Intercepting Docker</h1>

Run the following to intercept

{{< highlight text >}}
socat -v UNIX-LISTEN:/tmp/fake,fork UNIX-CONNECT:/var/run/docker.sock
{{< /highlight >}}

Run the following to make sure docker CLI send command to our proxy in /tmp/fake

{{< highlight text >}}
export DOCKER_HOST=unix:///tmp/fake
docker image list
{{< /highlight >}}


The output will show as follows

{{< highlight text >}}
> 2019/09/12 20:38:27.924464  length=81 from=0 to=80
HEAD /_ping HTTP/1.1\r
Host: docker\r
User-Agent: Docker-Client/xx.xx.1 (linux)\r
\r
< 2019/09/12 20:38:27.924817  length=287 from=0 to=286
HTTP/1.1 200 OK\r
Api-Version: 1.41\r
Cache-Control: no-cache, no-store, must-revalidate\r
Content-Length: 0\r
Content-Type: text/plain; charset=utf-8\r
Docker-Experimental: false\r
Ostype: linux\r
Pragma: no-cache\r
Server: Docker/library-import (linux)\r
Date: Thu, 12 Sep 2019 10:38:27 GMT\r
\r
> 2019/09/12 20:38:27.925517  length=92 from=81 to=172
GET /v1.40/images/json HTTP/1.1\r
Host: docker\r
User-Agent: Docker-Client/xx.xx.1 (linux)\r
\r
< 2019/09/12 20:38:27.926859  length=889 from=287 to=1175
HTTP/1.1 200 OK\r
Api-Version: 1.41\r
Content-Type: application/json\r
Docker-Experimental: false\r
Ostype: linux\r
Server: Docker/library-import (linux)\r
Date: Thu, 12 Sep 2019 10:38:27 GMT\r
Content-Length: 679\r
\r
[{"Containers":-1,"Created":1568087900,"Id":........

.....
.....
.....

{{< /highlight >}}


