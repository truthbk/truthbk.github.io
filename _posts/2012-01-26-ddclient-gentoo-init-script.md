---
layout: post
title: DDClient Gentoo init script
date: 2012-01-26 02:27:18.000000000 -05:00
categories:
- Linux
- Scripting
tags: []
status: publish
type: post
published: true
meta:
  dcssb_short_url: http://su.pr/1fKEyJ
  _syntaxhighlighter_encoded: '1'
  _edit_last: '2'
author:
  login: admin
  email: truthbk@gmail.com
  display_name: truth
  first_name: ''
  last_name: ''
---
Hello people, I know I've been absent for a while and I'm sorry about that. Then again, its not like I have legions following me, so my guess is no one has really noticed :)
Anyway, I have a nice few posts coming up. Particularly one regarding a small "fork" or mod of intel's igb NIC driver that allows parallel traffic capture from several processes from userspace with zero-copy and avoiding skbuff overhead. But I don't want to get ahead of myself, that will be coming up, together with the github(most likely) repo and explanation.

This post is really nothing special, I just hope it can help some gentoo users like myself. My ISP recently changed my service from fixed ip to dynamic ip (yup, I somehow managed to keep a fixed IP address for something like 8 years). So my [dyndns.org](http://dyndns.org) account needs a constant update of the DNS records. They suggest two alternatives [ddclient](http://dyn.com/support/clients/linux/ddclient/) and [inadyn](http://dyn.com/support/clients/linux/inadyn/), and as you can see in their links, they provide some decent literature and even config file generators, not bad. I went for ddclient, a perl based client. Ddclient comes ready with init script for debian, fedora, ubuntu,... users. But as you may know, gentoo users are not as common. Furthermore, init scripts are a little different in gentoo (I think in this case different is better). I want the ddclient daemon to start automatically at startup, so I needed/wanted the corresponding init.d gentoo script. I quickly hacked one up. Enjoy.

```
#!/sbin/runscript
# This file is part of ddclient for gentoo.
#
# Author: Jaime Fullaondo
#
CONF=/etc/ddclient/ddclient.conf
DDCLIENT="/usr/sbin/ddclient"
PID=`ps -aef | grep "$program - sleep" | grep -v grep | awk '{print $2}'`
depend() {
	need net
	need logger
}
start() {
	ebegin "Starting ddclient"
	if [ ! -d /var/run/ddclient ]; then
		mkdir /var/run/ddclient || return 1
	fi
	DELAY=`grep -v '^s*#' $CONF | grep -i -m 1 "daemon" | awk -F '=' '{print $2}'`
	if [ -z "$DELAY" ] ; then
        	DELAY="-daemon 300"
	else
		DELAY=''
	fi
	${DDCLIENT} $DELAY
	eend $? "Failed to start ddclient"
}
stop() {
	ebegin "Stopping ddclient"
	if [ -n "$PID" ] ; then
		kill $PID
	fi
	eend $? "Failed to stop ddclient"
}
restart() {
	if ! service_stopped "${SVCNAME}" ; then
		svc_stop || return "$?"
		sleep 1
	fi
	svc_start
}
```
