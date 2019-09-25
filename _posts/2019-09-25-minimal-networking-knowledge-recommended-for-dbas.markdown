---
layout: post
redirect_from:
  - 2019/09/25/minimal-networking-knowledge-recommended-for-dbas/
  - 2019/09/25/minimal-networking-knowledge-recommended-for-dbas.html
image: /assets/images/millsap_book.jpg

author: Fernando Ipar
status: publish
published: true
title: Minimal networking knowledge recommended for DBAs 
author: Fernando Ipar
author_login: fernando
author_email:
categories:
- notes
tags:
- notes
- pythian
---

*This article is a cross post from [Pythian's blog](https://blog.pythian.com/minimal-networking-knowledge-recommended-for-dbas/)*

In a [recent post](https://fernandoipar.com/notes/2019/09/05/top-8-things-every-database-practitioner-should-know.html), I summarized what I consider to be the minimal topics one should be familiar with in order to do technical work with databases. 

One thing I left out, except for it being implicitly included in “Hardware Basics” and “Operating Systems Basics”, is networking.  In this post, I’ll go through what I consider the minimal networking knowledge one needs to know when working with databases. 

# Protocol Concepts
While there are some niche network protocols in use (in my experience, when working with mainframes, for example), database practitioners may well spend their whole lives dealing mostly or only with TCP/IP, so that’s where I suggest focusing your learning efforts. 

A great guide to the basics is [“TCP/IP Illustrated, Vol 1: The Protocols”](http://www.kohala.com/start/tcpipiv1.html). While a bit old, the fundamentals are still relevant, and the book is very clearly written, and packed with tcpdump captures supporting the explanations. I think a good way to learn with it, while then checking out what has changed since it was written, is going to the [RFC index](https://www.rfc-editor.org/rfc-index.html) and checking out RFCs mentioned in the book for a protocol you’re studying. If more recent RFCs are available, they’ll be mentioned in the index with an “obsoleted-by” note. 

This is a vast field, and not all of it is of immediate relevance to a database practitioner, so I’m providing the following list of what I consider the core networking knowledge you’ll need to work with (and primarily to troubleshoot) databases: 

- TCP’s three-way handshake
- TCP connection termination
- The MSL and its relationship with connection termination and establishment
- Slow start
- Flow Control
- How ARP works
- How DNS works
- The MTU and Path MTU
- Routing, with emphasis on routing errors and their meaning (for example, what’s the difference between no route to host and a timeout?)
- Basics of UDP and what it does not provide when compared with TCP (if you’re troubleshooting something that uses it, like Galera)

# Networking Tools
I’ll mention tools available on GNU/Linux here because that’s the most popular OS by far for the databases I currently work with, but once you know the protocol basics, you’re just one quick Internet-search away from discovering which tool you must use on another OS (for example, entering “How to find out my IP in Windows” in Google gets me instructions on how to use ipconfig on the very first result returned). If you’re not on GNU/Linux but you are working on another OS that’s part of the Unix family tree, there are two useful resources that can help you: 

- The [Unix Rosetta Stone](http://bhami.com/rosetta.html), 
- The [USE Method’s Rosetta Stone](http://www.brendangregg.com/USEmethod/use-rosetta.html). 

You’ll need to become familiar with the following commands to find out which IP address the host you’re working has, how to change it, or add/remove an alias, and how to modify the routing table or the firewall rules. For this, I recommend reading the man pages for the following commands: 

- [ip](https://linux.die.net/man/8/ip) (You may find a lot of older resources online mentioning ifconfig, which is what I learned to use, but ip is what should be used now),
- [ethtool](https://linux.die.net/man/8/ethtool),
- [arp](https://linux.die.net/man/8/arp),
- [iptables](https://linux.die.net/man/8/iptables)/[ip6tables](https://linux.die.net/man/8/ip6tables),
- [netstat](https://linux.die.net/man/8/netstat) or [ss](https://linux.die.net/man/8/ss).

Sometimes, you’ll end up working on a puzzling database problem that involves the network, and you’ll want to know if the client is sending what you actually mean to send, the server is sending the response the client ends up seeing, etc. In those (and other) cases, you’ll want to capture network traffic and analyze it to see what is being sent over the pipe. 

For this, I recommend getting familiar with [tcpdump](https://www.tcpdump.org/) and [wireshark](https://www.wireshark.org/). I also recommend that you get familiar with these tools as you learn the protocols because it is an interrelated process: 

- You’ll read how the three-way handshake works.
- You’ll want to go and establish a TCP connection while capturing traffic, then analyze the traffic and see if it matches what the protocol says should happen. 

This is how the examples in the Stevens book I recommended earlier were made, but I think it’s priceless to come up with your own captures as you learn this. 

# Wrapping up

As database professionals, while we are specifically responsible for the health of database systems, our world does not end up there. Imagine a colleague (or client) that can’t connect to the database. You can’t just say, “The database logs show no connection attempts with your user; there’s nothing I can do.” You can instead get into a joint troubleshooting session, verifying that the client machine can reach the database server, at the right port, that a connection can be established, etc. When the problem is in the network, sometimes the solution is out of your hands (imagine a firewall sitting between you and the user, filtering traffic). But if you go to the network team with substantial evidence that shows how the connection can’t be established because it times out, despite the right routes being available, it will make their job easier, which in turns lets them be more effective at helping you resolve the problem. Great engineers always go the extra mile. Plus, troubleshooting network problems will be a welcome distraction from troubleshooting database problems!
