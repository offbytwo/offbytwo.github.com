---
layout: post
title: Show IP address of VM as console pre-login message
---

# {{ page.title }}

<p class="meta">9 May 2008 - Chicago</p>

In case you didn't know the pre-login message you see at a Linux console typically comes from /etc/issue

You can customize this file to alter the message with some escape codes that will show things like the current date and time, machine name and domain, kernel version, etc. But one thing you can't easily display is the IP address of a machine. Showing the IP address is especially useful when building a virtual machine that will use DHCP, like the Ubuntu development VM I use on my Macbook Pro. This way I can start VMware Fusion, see the IP address of the VM and then login over SSH.

In order to get the IP address to show in /etc/issue I needed to write a custom script that will rewrite /etc/issue with the IP address when the network interface is brought up. The first step was writing a simple script that will output the current IP address when run (by looking at the output of ifconfig).

<pre class="terminal"><code>/sbin/ifconfig | grep "inet addr" | grep -v "127.0.0.1" | awk '{ print $2 }' | awk -F: '{ print $2 }'</code></pre>

The above script will run ifconfig and print out the IP address (after filtering out the localhost interface). I saved this script to <code>/usr/local/bin/get-ip-address</code>. In order to get this into /etc/issue I decided to first copy <code>/etc/issue</code> to <code>/etc/issue-standard</code>, then create the following script that when run will overwrite /etc/issue with the contents of <code>/etc/issue-standard</code> + IP address.

### Debian/Ubuntu

Save the following script as <code>/etc/network/if-up.d/show-ip-address</code>

<pre><code>
#!/bin/sh
if [ "$METHOD" = loopback ]; then
    exit 0
fi

1. Only run from ifup.
if [ "$MODE" != start ]; then
    exit 0
fi

cp /etc/issue-standard /etc/issue
/usr/local/bin/get-ip-address >> /etc/issue
echo "" >> /etc/issue
</code></pre>

and don't forget to mark it executable.


### RedHat/CentOS

Save the following script as <code>/sbin/ifup-local</code>

<pre><code>
#!/bin/sh

if [ "$1" = lo ]; then
    exit 0
fi

cp /etc/issue-standard /etc/issue
/usr/local/bin/get-ip-address >> /etc/issue
echo "" >> /etc/issue
</code></pre>

and don't forget to mark it executable.
