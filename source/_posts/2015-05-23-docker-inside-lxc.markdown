---
layout: post
title: "docker Inside LXC"
date: 2015-05-23 02:08:23 +0530
comments: true
categories: docker lxc virtualization container 
---
Install lxc
{% codeblock Terminal %}
apt-get update && apt-get install lxc
{% endcodeblock %}

Change lxc default.conf
Add these line
{% codeblock Terminal %}
lxc.aa_profile = unconfined
lxc.cgroup.devices.allow = a
lxc.cap.drop =
{% endcodeblock %}
<!-- more -->

Nothing to list images, there is but in hidden way. just run `lxc-create -t download -n anyname`
{% codeblock Terminal %}
lxc-create -t download -n test01 -- -d ubuntu -r trusty -a amd64
{% endcodeblock %}

Start the image
{% codeblock Terminal %}
lxc-start -n test01 -d 
{% endcodeblock %}

Get status of the container
{% codeblock Terminal %}
lxc-ls --fancy
NAME      STATE    IPV4        IPV6  GROUPS  AUTOSTART
------------------------------------------------------
test01  RUNNING  10.0.3.225  -     -       NO
{% endcodeblock %}

Make apparamor changes [Note: This require only if you didn't updated the default.conf file ]
{% codeblock Terminal %}
# Distribution configuration
lxc.include = /usr/share/lxc/config/ubuntu.common.conf
lxc.arch = x86_64

# Container specific configuration
lxc.aa_profile = unconfined
lxc.cgroup.devices.allow = a
lxc.cap.drop =
lxc.rootfs = /var/lib/lxc/test01/rootfs
lxc.utsname = test01

# Network configuration
lxc.network.type = veth
lxc.network.link = lxcbr0
lxc.network.flags = up
lxc.network.hwaddr = 00:16:3e:6c:ee:91
{% endcodeblock %}

Install some packages inside container
{% codeblock Terminal %}
lxc-attach --name=test01
apt-get install wget apparmor => [Needed by docker, otherwise docker upstart will fail]
wget -qO- https://get.docker.com/ | sh
{% endcodeblock %}

Status of docker process
{% codeblock Terminal %}
root@test01:~# status docker
docker start/running, process 5113
{% endcodeblock %}

Check info
{% codeblock Terminal %}
root@test01:~# docker info
Containers: 0
Images: 0
Storage Driver: aufs
 Root Dir: /var/lib/docker/aufs
  Backing Filesystem: extfs
   Dirs: 0
    Dirperm1 Supported: false
    Execution Driver: native-0.2
    Kernel Version: 3.13.0-49-generic
    Operating System: Ubuntu 14.04.2 LTS (containerized)
    CPUs: 2
    Total Memory: 7.767 GiB
    Name: test01
    ID: ZVIJ:HC4N:T3AF:YFKN:JTXX:AUST:WJJG:GVZ7:YTVP:F5QZ:OYSK:HOFR
    WARNING: No swap limit support
{% endcodeblock %}

Check version
{% codeblock Terminal %}
root@test01:~# docker version
Client version: 1.6.2
Client API version: 1.18
Go version (client): go1.4.2
Git commit (client): 7c8fca2
OS/Arch (client): linux/amd64
Server version: 1.6.2
Server API version: 1.18
Go version (server): go1.4.2
Git commit (server): 7c8fca2
OS/Arch (server): linux/amd64
{% endcodeblock %}

Run a ubuntu container
{% codeblock Terminal %}
root@test01:~# docker run -t -i ubuntu /bin/bash
Unable to find image 'ubuntu:latest' locally
latest: Pulling from ubuntu
e9e06b06e14c: Pull complete
a82efea989f9: Pull complete
37bea4ee0c81: Pull complete
07f8e8c5e660: Already exists
ubuntu:latest: The image you are pulling has been verified. Important: image verification is a tech preview feature and should not be relied on to provide security.
Digest: sha256:8126991394342c2775a9ba4a843869112da8156037451fc424454db43c25d8b0
Status: Downloaded newer image for ubuntu:latest
root@44e667fbf162:/#
root@44e667fbf162:/#
root@44e667fbf162:/#
root@44e667fbf162:/# exit
exit
{% endcodeblock %}

Status of newly created image and container
{% codeblock Terminal %}
root@test01:~# docker images
REPOSITORY          TAG                 IMAGE ID            CREATED             VIRTUAL SIZE
ubuntu              latest              07f8e8c5e660        2 weeks ago         188.3 MB
root@test01:~# docker ps -a
CONTAINER ID        IMAGE               COMMAND             CREATED             STATUS                     PORTS               NAMES
44e667fbf162        ubuntu:latest       "/bin/bash"         24 seconds ago      Exited (0) 7 seconds ago                       modest_yonath
{% endcodeblock %}

Any problem, look here to troubleshoot
{% codeblock Terminal %}
tail -f /var/log/kern.log /var/log/syslog /var/log/dmesg /var/log/upstart/docker.log
{% endcodeblock %}
