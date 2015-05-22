---
layout: post
title: "Apache 2.4 with Mod_QOS"
date: 2014-07-16 13:01:49 +0530
comments: true
categories: [ apache, "mod_qos" ]
---

Just a basic info about qos.

1. Create policy based on number of request or on bandwidth and lot more.
2. Event request controlling Mechanism
3. Mitigate DDoS attacks.

[More can be found over here](http://opensource.adnovum.ch/mod_qos/)
<!-- more -->

Current working enviornment is OpenSuSE 13.1 with Apache 2.4

Install Apache 
{% codeblock Terminal lang:bash %}
zypper install apache2 apache2-worker
{% endcodeblock %}

Download latest mod_qos [from this location](http://sourceforge.net/projects/mod-qos/files/)
{% codeblock Terminal lang:bash %}
tar xvf mod_qos-11.4.tar.gz
cd mod_qos-11.4
cd mod_qos-11.4/apache2 && /usr/sbin/apxs2 -I /usr/include/apache2-worker/ -ci mod_qos.c"
/usr/lib64/apr-1/build/libtool --silent --mode=compile gcc -std=gnu99 -prefer-pic -fmessage-length=0 -grecord-gcc-switches -fstack-protector -O2 -Wall -D_FORTIFY_SOURCE=2 -funwind-tables -fasynchronous-unwind-tables -g -fPIC -Wall -DLDAP_DEPRECATED  -DLINUX -D_REENTRANT -D_GNU_SOURCE -pthread -I/usr/include/apache2  -I/usr/include/apr-1   -I/usr/include/apr-1 -I/usr/include -I/usr/include/apache2-worker/  -c -o mod_qos.lo mod_qos.c && touch mod_qos.slo
/usr/lib64/apr-1/build/libtool --silent --mode=link gcc -std=gnu99    -o mod_qos.la  -rpath /usr/lib64/apache2 -module -avoid-version    mod_qos.lo
/usr/share/apache2/build/instdso.sh SH_LIBTOOL='/usr/lib64/apr-1/build/libtool' mod_qos.la /usr/lib64/apache2
/usr/lib64/apr-1/build/libtool --mode=install install mod_qos.la /usr/lib64/apache2/
libtool: install: install .libs/mod_qos.so /usr/lib64/apache2/mod_qos.so
libtool: install: install .libs/mod_qos.lai /usr/lib64/apache2/mod_qos.la
libtool: finish: PATH="/sbin:/usr/sbin:/usr/local/sbin:/root/bin:/usr/local/bin:/usr/bin:/bin:/usr/X11R6/bin:/usr/games:/opt/bin:/sbin" ldconfig -n /usr/lib64/apache2
----------------------------------------------------------------------
Libraries have been installed in:
   /usr/lib64/apache2

If you ever happen to want to link against installed libraries
in a given directory, LIBDIR, you must either use libtool, and
specify the full pathname of the library, or use the `-LLIBDIR'
flag during linking and do at least one of the following:
   - add LIBDIR to the `LD_LIBRARY_PATH' environment variable
     during execution
   - add LIBDIR to the `LD_RUN_PATH' environment variable
     during linking
   - use the `-Wl,-rpath -Wl,LIBDIR' linker flag
   - have your system administrator add LIBDIR to `/etc/ld.so.conf'

See any operating system documentation about shared libraries for
more information, such as the ld(1) and ld.so(8) manual pages.
----------------------------------------------------------------------
chmod 755 /usr/lib64/apache2/mod_qos.so
{% endcodeblock %}

If you see the Output like this that means mod_qos is installed and you just have to load it in apache.

We will add a line in this file `/etc/apache2/sysconfig.d/loadmodule.conf`
{% codeblock Terminal lang:bash %}
LoadModule qos_module                     /usr/lib64/apache2/mod_qos.so
{% endcodeblock %}

You can load mod_qos for all the virtualhost or to any specific virtualhost. I'm adding to a specific one.

{% codeblock Terminal lang:bash %}
<VirtualHost *:80>
    ServerName        www.example.com
    DocumentRoot      /web/example.com/
    ServerAdmin       webmaster@example.com
    ErrorLog          /var/log/apache2/example.com-error_log
    TransferLog       /var/log/apache2/example.com-access_log
    ServerSignature   Off
    HostnameLookups   Off
    Options           -Indexes -FollowSymLinks
    #-------------------------
    <Directory "/web/example.com">
        Allow        from all
    </Directory>
    <IfModule mod_qos.c>
        <Location /qos>
           SetHandler qos-viewer
        </Location>
    </IfModule>
</VirtualHost>
{% endcodeblock %}

You can check the status of mod_qos on this link. **NOTE** change the domainname.

`http://domain.name.com/qos?option=ip&action=enable&refresh=`

