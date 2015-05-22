---
layout: post
title: "How to setup Vagrant"
date: 2014-07-01 12:38:48 +0530
comments: true
categories: [vagrant, virtualization, puppet]
---

What is Vagrant ?

The big question, which I was thinking, when I first heard of it. So today finally got a chance to do some hands on.
Basically it is a tools which gives you running VM, whenever you need it with all your environment set in place.
So what was that huh? Consider that you are creating a VM or clonning a VM and starting it up and running some script to get 
the environment set in place. So why not automate this whole thing by vagrant. 

Vagrant allows you to launch VMs with provisioning on top of it.

<!-- more -->
![Image](https://raw.githubusercontent.com/ashish1099/ashish1099.github.io/master/images/blog_images/logo_vagrant.png)

**Some Nomenclature**

*`box`* : Base image of an OS, [Read here](https://docs.vagrantup.com/v2/boxes.html)

*`Provider`* : Hyperviosr in which VMs are going to run. These are Virtualbox, VMware and many more. [Read here](https://docs.vagrantup.com/v2/providers/index.html)

*`Provising`* : Tools like Puppet, Chef, Ansibal and ofcourse Shell script. [Read here](https://docs.vagrantup.com/v2/provisioning/index.html)

`Working environment is Ubuntu 12.04`

Install this packages
{% codeblock Terminal lang:bash %}
sudo apt-get install dpkg-dev virtualbox linux-headers-generic linux-image-generic linux-generic vagrant
{% endcodeblock %}

Know the Virtualbox and Vagrant is installed, we will add `box` in vagrant.
{% codeblock Terminal lang:bash %}
vagrant box add precise32 http://files.vagrantup.com/precise32.box
{% endcodeblock %}

It will take some time, and depends on your bandwidth. After the download is completed, there will be a `Vagrantfile` in the working directory.
{% codeblock Terminal lang:bash %}
ajaiswal@c-0242:~$ ls -l
-rw-rw-r--  1 ajaiswal ajaiswal 3876 Jul  1 12:31 Vagrantfile
{% endcodeblock %}

Open the Vagrantfile and change this line to
{% codeblock Terminal lang:bash %}
config.vm.box = "precise32"
{% endcodeblock %}

You can list the boxes, like this
{% codeblock Terminal lang:bash %}
ajaiswal@c-0242:~$ vagrant box list
precise32
{% endcodeblock %}

Now we can start the box with this command.
{% codeblock Terminal lang:bash %}
ajaiswal@c-0242:~$ vagrant up
[default] Importing base box 'precise32'...
[default] The guest additions on this VM do not match the install version of
VirtualBox! This may cause things such as forwarded ports, shared
folders, and more to not work properly. If any of those things fail on
this machine, please update the guest additions and repackage the
box.

Guest Additions Version: 4.2.0
VirtualBox Version: 4.1.12
[default] Matching MAC address for NAT networking...
[default] Clearing any previously set forwarded ports...
[default] Forwarding ports...
[default] -- 22 => 2222 (adapter 1)
[default] Creating shared folders metadata...
[default] Clearing any previously set network interfaces...
[default] Booting VM...
[default] Waiting for VM to boot. This can take a few minutes.
[default] VM booted and ready for use!
[default] Mounting shared folders...
[default] -- v-root: /vagrant
{% endcodeblock %}

Then we will login into the box.
{% codeblock Terminal lang:bash %}
ajaiswal@c-0242:~$ vagrant ssh
Welcome to Ubuntu 12.04 LTS (GNU/Linux 3.2.0-23-generic-pae i686)

 * Documentation:  https://help.ubuntu.com/
Welcome to your Vagrant-built virtual machine.
Last login: Fri Sep 14 06:22:31 2012 from 10.0.2.2
{% endcodeblock %}

Running some command under the box.

{% codeblock Terminal lang:bash %}
vagrant@precise32:~$ ll
total 48
drwxr-xr-x 4 vagrant vagrant 4096 Sep 14  2012 ./
drwxr-xr-x 3 root    root    4096 Sep 14  2012 ../
-rw------- 1 vagrant vagrant  114 Sep 14  2012 .bash_history
-rw-r--r-- 1 vagrant vagrant  220 Sep 14  2012 .bash_logout
-rw-r--r-- 1 vagrant vagrant 3486 Sep 14  2012 .bashrc
drwx------ 2 vagrant vagrant 4096 Sep 14  2012 .cache/
-rwxr-xr-x 1 vagrant vagrant 6487 Sep 14  2012 postinstall.sh*
-rw-r--r-- 1 vagrant vagrant  675 Sep 14  2012 .profile
drwx------ 2 vagrant vagrant 4096 Sep 14  2012 .ssh/
-rw-r--r-- 1 vagrant vagrant    0 Sep 14  2012 .sudo_as_admin_successful
-rw------- 1 vagrant vagrant    6 Sep 14  2012 .vbox_version
-rw------- 1 vagrant vagrant   12 Sep 14  2012 .veewee_version
{% endcodeblock %}

Hope this clear your basic doubts, Feel free if there is any question.
