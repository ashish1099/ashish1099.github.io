---
layout: post
title: "Collectd DRBD Plugin"
date: 2014-06-27 12:20:30 +0530
comments: true
categories: [bash, scripting, graphite]
---
I have ran into a scenario where my manager wanted to monitor drbd resource. So we are using graphite over here and pushing data from collectd to graphite. 
I did a quick search on **google.com**, but didn't find any collectd drbd plugin. Actually I found one which is in python, [This is the link](https://git.kumina.nl/packages/collectd-plugins-kumina/blob/master/drbd.py)

Only problem which I remember, when I was testing this, it looks for only one resource. Since I haven't learned python.
I thought of writing my own. I wrote a bash script and got the job done.

<!-- more -->

{% codeblock drbd.sh lang:bash %}
#!/bin/bash
# Example
# ns:0 nr:1950024 dw:1950024 dr:0 al:0 bm:143 lo:0 pe:0 ua:0 ap:0 ep:1 wo:d oos:0
# Author: Ashish Jaiswal

HOSTNAME=$(hostname)
INTERVAL=1

while sleep "$INTERVAL"
do
        # How many resource are running out there.
        resource_count=$(drbd-overview  | wc -l)
        for i in $(seq 1 $resource_count)
        do
                
                # Gives the list of the resource name
                name=$(drbd-overview | awk "NR==$i" | awk -F ":" '{print $2}' | cut -d "/" -f 1)
                
                ns=$(grep -A 1 "$i: cs:Connected" /proc/drbd | awk 'NR==2' | awk '{print $1}' | cut -d ":" -f 2)
                nr=$(grep -A 1 "$i: cs:Connected" /proc/drbd | awk 'NR==2' | awk '{print $2}' | cut -d ":" -f 2)
                dw=$(grep -A 1 "$i: cs:Connected" /proc/drbd | awk 'NR==2' | awk '{print $3}' | cut -d ":" -f 2)
                dr=$(grep -A 1 "$i: cs:Connected" /proc/drbd | awk 'NR==2' | awk '{print $4}' | cut -d ":" -f 2)
                al=$(grep -A 1 "$i: cs:Connected" /proc/drbd | awk 'NR==2' | awk '{print $5}' | cut -d ":" -f 2)
                bm=$(grep -A 1 "$i: cs:Connected" /proc/drbd | awk 'NR==2' | awk '{print $6}' | cut -d ":" -f 2)
                lo=$(grep -A 1 "$i: cs:Connected" /proc/drbd | awk 'NR==2' | awk '{print $7}' | cut -d ":" -f 2)
                pe=$(grep -A 1 "$i: cs:Connected" /proc/drbd | awk 'NR==2' | awk '{print $8}' | cut -d ":" -f 2)
                ua=$(grep -A 1 "$i: cs:Connected" /proc/drbd | awk 'NR==2' | awk '{print $9}' | cut -d ":" -f 2)
                ap=$(grep -A 1 "$i: cs:Connected" /proc/drbd | awk 'NR==2' | awk '{print $10}' | cut -d ":" -f 2)
                ep=$(grep -A 1 "$i: cs:Connected" /proc/drbd | awk 'NR==2' | awk '{print $11}' | cut -d ":" -f 2)
                oss=$(grep -A 1 "$i: cs:Connected" /proc/drbd | awk 'NR==2' | awk '{print $13}' | cut -d ":" -f 2)

                echo "PUTVAL $HOSTNAME/drbd-$name/network_send interval=$INTERVAL N:$ns"
                echo "PUTVAL $HOSTNAME/drbd-$name/network_receive interval=$INTERVAL N:$nr"
                echo "PUTVAL $HOSTNAME/drbd-$name/disk_write interval=$INTERVAL N:$dw"
                echo "PUTVAL $HOSTNAME/drbd-$name/disk_read interval=$INTERVAL N:$dr"
                echo "PUTVAL $HOSTNAME/drbd-$name/activity_log interval=$INTERVAL N:$al"
                echo "PUTVAL $HOSTNAME/drbd-$name/bit_map interval=$INTERVAL N:$bm"
                echo "PUTVAL $HOSTNAME/drbd-$name/local_count interval=$INTERVAL N:$lo"
                echo "PUTVAL $HOSTNAME/drbd-$name/pending interval=$INTERVAL N:$pe"
                echo "PUTVAL $HOSTNAME/drbd-$name/unacknowledged interval=$INTERVAL N:$ua"
                echo "PUTVAL $HOSTNAME/drbd-$name/application_pending interval=$INTERVAL N:$ap"
                echo "PUTVAL $HOSTNAME/drbd-$name/epochs interval=$INTERVAL N:$ep"
                echo "PUTVAL $HOSTNAME/drbd-$name/out_of_sync interval=$INTERVAL N:$oss"
        done # End of name
done # End of while
{% endcodeblock %}

Create a file under `/etc/collect.d/drbd` and enter this
{% codeblock %}
LoadPlugin exec
<Plugin exec>
Exec "nobody" "/usr/lib64/collectd/python-plugins/drbd.sh"
NotificationExec "nobody" "/usr/lib64/collectd/python-plugins/drbd.sh"
</Plugin>
{% endcodeblock %}
