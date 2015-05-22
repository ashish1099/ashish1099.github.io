---
layout: post
title: "DRBD automatic failover without cluster"
date: 2014-12-26 13:54:19 +0530
comments: true
categories: drbd bash script 
---

I have written a script to do a failover of DRBD resources. This script will get the drbd start mount the partition,
and start the nfsserver and samba server and the get virtual IP setup.

This script won't be useful for many, which are having a mix envrionment. such as node having some resource as primary and some as secondary.
Reason behind this, that we where not able to use pacemaker in our current enviornment. Though we are not using this in production enviornment.

Put this in your systemd bootup process. [check this](https://wiki.archlinux.org/index.php/Systemd_FAQ#How_can_I_make_a_script_start_during_the_boot_process.3F)

Please use it and let me know.
<!-- more -->

{% codeblock drbd_auto.sh lang:bash %}

#!/bin/bash
# Ashish Jaiswal
# Date 16th Dec 2014
# This script expects the primary host to be primary node
# for all the drbd resources.

# List of resource on the node.
res=$(cat /etc/drbd.d/*.res | grep resource | cut -d " " -f 2)

# eth1 is the interface which we going to assign floating IP
device_id=eth1

# This is the floating IP
ip="192.168.1.100"

function services () {
  # Declaring an array of the services, we want to be stop or start
  service_name=(nfsserver smb nmb)

  for var in ${service_name[@]}
  do
    systemctl is-active $var | grep ^active > /dev/null

    if [ $? -ne 0 ]; then
      systemctl $1 $var
    fi
  done
}

function check_ip {
  # Check whether the ip is present or not
  primary_ip=$(ip -o -4 addr show $device_id | awk -F '[ /]+' '/global/ {print $4}')

  # Add ip if is not present 
  if [ -z $primary_ip ]; then
   ip address add $ip dev $device_id
  fi
}

function mount_status {
  mountpoint -q /$1
  if [ $? -ne 0 ]; then
      mount /$1
  fi
}

function drbd_failover {
  drbdadm primary --force $1
  if [ $? -ne 0 ]; then
    echo -e "There seems to be some major problem,\nPlease login to `hostname` and fix the error." | mail -s "DRBD failover Unsuccessful"  ashish1099@gmail.com 
    exit 1
  fi
}

for resource in $res; do
  # To get the current state of the resources.
  pri_roles=$(drbdadm role $resource  | cut -d "/" -f 1 | egrep "Unknown|Primary|Secondary")
  sec_roles=$(drbdadm role $resource  | cut -d "/" -f 2 | egrep "Unknown|Primary|Secondary")

  # If the node has a primary resources
  if [ $pri_roles = Primary ] 
  then
    mount_status $resource
    services start
    check_ip
    echo "This is primary node for $resource resource on `hostname`"
	fi

  # If the node has a secondary resources
  if [ $pri_roles = Secondary ] || [ $pri_roles = Unknown ]
  then
    ping -c2 $ip > /dev/null && mountpoint -q /$resource
    if [ $? -ne 0 ] 
    then
      if [ $sec_roles = Primary ]
      then
        echo "$resource is available on Primary, This is a secondary node."
      fi

      if [ $sec_roles = Unknown ]
      then
        drbd_failover $resource
        mount_status $resource
        services start
        check_ip 
        logger "Successfully moved drbd resources $resource and started nfs and samba on `hostname`"
        echo "Successfully moved drbd resources $resource and started nfs and samba on `hostname`" | mail -s "DRBD Successful Failover"  ashish1099@gmail.com
      fi

      if [ $sec_roles = Secondary ]
      then
        echo -e "Split brain detected, Please fix manually\n\nAll these command need to be ran\n# drbdadm secondary <resource>\n# drbdadm connect --discard-my-data <resource>\n\nOn the other node (the split brain survivor), if its connection state is also StandAlone\n# drbdadm connect <resource>\n\nHere is drbd doc link http://www.drbd.org/users-guide/s-resolve-split-brain.html" | mail -s "DRBD SplitBrain Detected"  ashish1099@gmail.com
      fi
    else
      services stop
      logger "This is seconday node of $resource. So nothing to be done on this server, because Primary is Active at this moment"
    fi
  fi
done
{% endcodeblock %}
