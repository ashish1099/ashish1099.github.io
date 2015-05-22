---
layout: post
title: "Convert CentOS/Scientific Linux to RHEL server"
date: 2014-09-19 15:54:26 +0530
comments: true
categories: [ linux ]
---

We have ran into scenario where we wanted to convert a CentOS box to RHEL box.
So sharing my experience out here.

There are some pre-requisite 
{% codeblock Terminal lang:bash %}
rpm -qa centos\*
centos-indexhtml-6-1.el6.centos.noarch
centos-release-6-5.el6.centos.11.2.x86_64
{% endcodeblock %}

<!-- more -->
We will need some redhat packages to start off. You can retrive this packages from installation media or you can logging to RHN and download these packages.

{% codeblock Terminal lang:bash %}
libgudev1-147-2.51.el6.x86_64.rpm
pygobject2-2.20.0-5.el6.x86_64.rpm
redhat-release-server-6Server-6.5.0.1.el6.x86_64.rpm
rhn-setup-1.0.0.1-8.el6.noarch.rpm
yum-rhn-plugin-0.9.1-49.el6.noarch.rpm
m2crypto-0.20.2-9.el6.x86_64.rpm
python-dmidecode-3.10.13-3.el6_4.x86_64.rpm
rhn-check-1.0.0.1-8.el6.noarch.rpm
rhnlib-2.5.22-15.el6.noarch.rpm
pyOpenSSL-0.10-2.el6.x86_64.rpm
python-gudev-147.1-4.el6_0.1.x86_64.rpm
rhn-client-tools-1.0.0.1-8.el6.noarch.rpm
rhnsd-4.9.3-2.el6.x86_64.rpm
{% endcodeblock %}

1. First we will remove CentOS packages

{% codeblock Terminal lang:bash %}
	rpm -e --nodeps centos-release centos-indexhtml pygobject2 pyOpenSSL libgudev1
{% endcodeblock %}

2. Install those redhat packages, I had these packages at `/root/rhel`, after installing those packages we will clean yum cache.

{% codeblock Terminal lang:bash %}
	cd /root/rhel
	rpm -ivh *
	yum clean all
{% endcodeblock %}

3. Then you will have to register the box to RHN 

{% codeblock Terminal lang:bash %}
rhn_register
cat /etc/redhat-release
{% endcodeblock %}

4. Just assigning an optional channel to RHEL box.
{% codeblock Terminal lang:bash %}
rhn-channel -a -c rhel-x86_64-server-optional-6
{% endcodeblock %}

5. Just run an update, to get updates from RedHat Network
{% codeblock Terminal lang:bash %}
yum update
{% endcodeblock %}

6. The below command, will put all the other packages names with version number, and we will reinstall those with RedHat repository
{% codeblock Terminal lang:bash %}
rpm -qa --qf "%{NAME} %{VENDOR}\n" | grep CentOS | cut -d' ' -f1 | grep -v ^kernel | sort | tee lst
yum reinstall $(cat lst)
{% endcodeblock %}

7. We will run a distro-sync for the left out package.
{% codeblock Terminal lang:bash %}
yum distro-sync
{% endcodeblock %}

8. Just to ensure if there is any left out package which is not reinstall from RedHat repos. In some case there maybe some, just subscribe to those required channels and your are good to go.
Then just reboot the box. 
{% codeblock Terminal lang:bash %}
rpm -qa --qf "%{NAME} %{VENDOR}\n" | grep CentOS | cut -d' ' -f1 | grep -v ^kernel | wc -l
0
yum update
No Packages marked for Update
yum repolist
{% endcodeblock %}




http://blog.famillecollet.com/post/2013/09/21/Switch-from-CentOS-6.4-to-RHEL-6.4
