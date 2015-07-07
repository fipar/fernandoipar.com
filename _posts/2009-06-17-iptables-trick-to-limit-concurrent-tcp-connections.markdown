---
layout: post
redirect_from:
  - 2009/06/17/iptables-trick-to-limit-concurrent-tcp-connections/

author: Fernando Ipar
status: publish
published: true
title: iptables trick to limit concurrent tcp connections
author_login: fernando
author_email: 
wordpress_id: 173
date: !binary |-
  MjAwOS0wNi0xNyAwOTo1ODoxMCAtMDMwMA==
date_gmt: !binary |-
  MjAwOS0wNi0xNyAxMTo1ODoxMCAtMDMwMA==
categories:
- MySQL
tags:
- security
- MySQL
- concurrency
- bugs
comments:
- id: 76
  author: Edward Hall
  author_email: EdwardLHall@hotmail.com
  author_url: ''
  date: !binary |-
    MjAwOS0wOC0xOCAxMDowMToyNSAtMDMwMA==
  date_gmt: !binary |-
    MjAwOS0wOC0xOCAxMzowMToyNSAtMDMwMA==
  content: ! "Very Nice!\r\n\r\nI have been trying to find a way to prevent Apache
    from spawning too many child processes whenever some bad robot or hacker throws
    a bunch of concurrent requests at our website.\r\n\r\nThis may just do the trick..."
- id: 77
  author: fernando
  author_email: mail@fernandoipar.com
  date: !binary |-
    MjAwOS0wOC0xOCAxMDoyMToxNSAtMDMwMA==
  date_gmt: !binary |-
    MjAwOS0wOC0xOCAxMzoyMToxNSAtMDMwMA==
  content: ! "Glad you found it useful!\r\n\r\nThis might be of interest to you too:
    http://www.cohprog.com/mod_bandwidth.html, though I don't know if it's still active,
    or if it works across apache v 1.x and 2.x. \r\n\r\nThe good side of the iptables
    approach is that it can be used to limit concurrent connections to any network
    based service\r\n\r\nGood luck,"
- id: 139
  author: antonio
  author_email: apedic@netsystemsa.it
  author_url: http://www.netsystemsa.it
  date: !binary |-
    MjAxMC0wOS0yNCAxNjoyNDo1MCAtMDMwMA==
  date_gmt: !binary |-
    MjAxMC0wOS0yNCAxOToyNDo1MCAtMDMwMA==
  content: it is possible to see the "bad list" ?
---
<p>This is sort of a self-documenting post, and a self-support group about ill-behaved tomcat apps.</p>
<p>Sometimes, you have multiple nodes accesing your MySQL server (or any kind of server, for that matter) concurrently. Eventually, software in one or more of these nodes might do nasty things (you know who you are buddy:))</p>
<p>MySQL provides a built in mechanism to limit concurrent connections, but this can only be set for the whole server, or on a per user basis. Unfortunatly, most of these setups use the same database user for all their nodes, so this feature can't be used to confine any possible damage.</p>
<p>Enter your good friend iptables.</p>
<p>This isn't perfect, but this little trick might help you while programmers take care of their business:</p>
<pre>iptables -A INPUT -p tcp -m recent --rcheck --seconds 60 -j REJECT
iptables -A INPUT -p tcp --dport 3306 -m connlimit --connlimit-above 2 -m recent --set -j REJECT</pre>
<p>(The number of seconds and the concurrency limit here are examples for testing only, set them to proper values if you use them in your servers!)</p>
<p>This two rules create a recent 'bad guy' list, and send any source that exceeds two concurent connections on tcp pot 3306 to this list for 60 seconds.</p>
<p>If used smartly with a proper timeout value for MySQL connections, this could be useful for situations such as the one I described.</p>
<p>Hope it helps you!</p>
