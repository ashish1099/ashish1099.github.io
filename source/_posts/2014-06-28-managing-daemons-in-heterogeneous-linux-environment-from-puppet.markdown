---
layout: post
title: "Managing daemons in heterogeneous linux environment from puppet"
date: 2014-06-28 00:01:13 +0530
comments: true
categories: [puppet]
---
We have a lot of box mixed with heterogeneous linux environment. So managing a daemons is a bit struggling. So worte my own manifest to over come this. I don't know what other options there are, But this seems to kind of work for me.
This can be scale upto **N** number of linux flavours, currently I have the manifest including only `CentOS, OpenSuSE, Ubuntu`. 


Let me know if you have any trouble with it or any improvement is always welcome. 

<!-- more -->
{% codeblock services.pp lang:puppet %}
class services {

        define daemons( 
			$service_ensure,
			$service_enable,
			$service_require = false
                ) {    

		$daemon_provider = $operatingsystem ? { 'OpenSuSE' => 'systemd', 'CentOS' => 'redhat', 'Ubuntu' => 'upstart' }
		
		if $service_require {
			Service { require => $daemon_require }
		}

                service { "$name" :
                        ensure => $service_ensure,
                        enable => $service_enable,
                        provider => $daemon_provider,
                }       

	} #End of daemons

	case $operatingsystem {
		"OpenSuSE" : {
	# OpenSUSE
	daemons { 'avahi-daemon' : 	service_ensure => stopped, service_enable => false }
	daemons { 'bluetooth.service' : service_ensure => stopped, service_enable => false }
	
	daemons { 'atd' : 		service_ensure => running, service_enable => true, service_require => "Package['at']"  }
	daemons { 'rpcbind' : 		service_ensure => running, service_enable => true, service_require => "Package['rpcbind']" }
		} #End of OpenSuSe
	
		"CentOS" : {
	# CentOS
	daemons { 'auditd' : 		service_ensure => stopped, service_enable => false }
	daemons { 'mdmonitor' : 	service_ensure => stopped, service_enable => false }

	daemons { 'nscd' : 		service_ensure => running, service_enable => true, service_require => "Package['nscd']" }
	daemons { 'ntpd' : 		service_ensure => running, service_enable => true, service_require => "Package['ntp']" }
		} #End of CentOS
	
		"Ubuntu" : { 
	# Ubuntu
	daemons { 'ypbind' : 		service_ensure => running, service_enable => true, service_require => "Package['nis']" }
	daemons { 'ssh' : 		service_ensure => running, service_enable => true, service_require => "Package['openssh-server']" }
		} # End of Ubuntu

	} #End of operatingsystem
} #End of baseos::ervices
{% endcodeblock %}

You can call this defination from any other module like this. I'm calling from my vsftp module.

{% codeblock vsftp.pp lang:puppet %}
services::daemons { 'vsftpd' :  service_ensure => running, service_enable => true, service_require => "Package['vsftpd']" } 
{% endcodeblock %}

