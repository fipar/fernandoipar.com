---
layout: post
redirect_from:
  - 2009/01/28/rewriting-highbase-in-erlang/

author: Fernando Ipar
status: publish
published: true
title: Rewriting Highbase in Erlang
author_login: fernando
author_email: 
wordpress_id: 70
date: !binary |-
  MjAwOS0wMS0yOCAwMToyODo0MiAtMDIwMA==
date_gmt: !binary |-
  MjAwOS0wMS0yOCAwMzoyODo0MiAtMDIwMA==
categories:
- Highbase
tags:
- Programming
- MySQL
- Highbase
- Erlang
comments:
- id: 5
  author: Frank Mueller
  author_email: mue@tideland.biz
  author_url: http://mue.tideland.biz
  date: !binary |-
    MjAwOS0wMS0yOSAwNjo1Mjo1MCAtMDIwMA==
  date_gmt: !binary |-
    MjAwOS0wMS0yOSAwODo1Mjo1MCAtMDIwMA==
  content: Hi Fernando, before start using gen_server you should take a deeper look
    into gen_fsm. It's a behaviour for finite state machines as you've described above.
    And it's as easy and powerful as gen_server. Regards, mue
- id: 6
  author: fernando
  author_email: mail@fernandoipar.com
  date: !binary |-
    MjAwOS0wMS0yOSAxMDo0MDozMSAtMDIwMA==
  date_gmt: !binary |-
    MjAwOS0wMS0yOSAxMjo0MDozMSAtMDIwMA==
  content: ! "Hi Frank, \r\n\r\nThanks for your recommendation. \r\nI did read a couple
    of examples of gen_fsm and yes, it is better for this design, I guess I put it
    in the back of my mind, so it's good to have someone more experienced in Erlang/OTP
    remind me about it :)\r\n\r\nI'm still trying to put all the pieces together,
    I'm definitely trying to make a good use of the event handling mechanism. \r\n\r\nGoing
    through Armstrong's book for, like, the fourth time now :)\r\n\r\nRegards, Fernando."
---
<h1>Why?</h1>
<p><a title="Highbase : High availability for MySQL" href="http://highbase.seriema-systems.com">Highbase</a> is currently comprised of several shell scripts  and some C code. It's actually a good project (talk about self promotion) that hasn't reached a stable release yet just because</p>
<ol>
<li>It hasn't been tested enough in production environments</li>
<li>I've been amazingly busy during the last years. Lots of work, and lots of parenting in the last two and a half years in particular.</li>
</ol>
<p>Still, it's real close to a release candidate, and I will continue testing and fixing the main branch. However, I believe writing a new version in <a title="Erlang. Concurrent, distributed, fault tolerant programming" href="http://www.erlang.org">Erlang</a> has many advantages:</p>
<ol>
<li>Erlang was designed from the ground up with reliability in mind (among other things. It's quite popular now due to it's multi-core-friendlyness, but that's not really useful for a project like this), so it's only natural to use it to develop a high availability solution (you know, the "best tool for the job" mentality)</li>
<li>Except for the mysql service verification (I wouldn't want to go with odbc), and perhaps gratuitious ARP (I'm not sure at this point), rewriting in Erlang would remove all other dependencies in third party packages. In general, it would make the solution more mantainable and hence more reliable (see point 1).</li>
</ol>
<p>I already wrote a first draft in Erlang about a year ago, but this was when I was just learning the language, so I didn't take advantage of it's best features for reliability. Actually, the problem was I wrote my version using 'pure' Erlang, dismissing the OTP part. Erlang as a language is OK (it has anything you need to write reliable,concurrent software). Erlang/OTP is a powerful thing, in the sense that it abstracts you, as a programmer, from the low level chores of distributed, concurrent and fault tolerant programming. You just have to focus on your programming logic, and by following some conventions (Behaviours, which for OO people could loosely be related to Interfaces and Base Clases) you get a lot of extra functionality for free.</p>
<h1>How?</h1>
<p>This new rewrite will be done using the gen_server behaviour, together with events. All this free to use from OTP.</p>
<p>Here's a state diagram with an initial draft of the slave routine (done with <a title="Visual Paradigm. Modeling tools. " href="http://www.visual-paradigm.com/">Visual Paradigm</a>):</p>
<div class="mceTemp">
<dl id="attachment_69" class="wp-caption alignnone" style="width: 676px;">
<dt class="wp-caption-dt"></dt>
<dd class="wp-caption-dd">Highbase's state diagram</dd>
<dt class="wp-caption-dt"><a href="http://fernandoipar.com/wp-content/uploads/2009/01/highbase.jpg"><img class="size-full wp-image-69" title="State diagram" src="http://fernandoipar.com/wp-content/uploads/2009/01/highbase.jpg" alt="Highbase's state diagram" width="666" height="416" /></a></dt>
</dl>
</div>
<p><strong></strong></p>
<p>Without reading a line of code, here you can get a grasp of the power of OTP. The slave routine is, in it's normal state, just waiting for an event. So far, I've identified the following events:</p>
<ul>
<li>service down</li>
<li>link down (erlang link down, would probably be due to the master node being down)</li>
<li>shutdown request</li>
</ul>
<p>In the first case, just as today, we first attempt to restart the service, and if this fails, we go for a takeover. If restart fails, we first to a failover (i.e., shutdown of the master node, this is the same algorithm executed on our <a title="Highbase - the current trunk" href="http://highbase.svn.sourceforge.net/viewvc/highbase/trunk/highbase/">current code branch</a>).</p>
<p>In the second case, if the node is down, it's just a takeover. If the node is up, we do a takeover with ARP spoofing, because we assume there's something weird with the other node. This part of the algorithm can be improved (we could do another verification, go back to the waiting for event state for N times, etc., this is just a draft).</p>
<p>Still, one of the improvements over the current version is that I don't have to handle any loops, OTP handles that for me, all I have to do is write the callback functions (waiting_for_event, restarting_service, etc). Less code = less chance of errors.</p>
<h1><strong>When?</strong></h1>
<p>I'm currently revewing the design of the new algorithm. I hope to have a draft in two weeks (this is a slow week for me, my daughter is starting preschool education next week and hence my house is kinda crazy).</p>
<p>I'm also overdue with regular Highbase releases so I'll try to get up to date with both trees.</p>
<p>The current erlang tree is <a title="Highbase: the old erlang tree" href="http://highbase.svn.sourceforge.net/viewvc/highbase/trunk/mysql-ha-erl/">mysql-ha-erl</a> (legacy name, before the trademark issues). I'll surely be creating a new tree, highbase-erl, during the next few days. Official release news will be broadcasted here from the official site.</p>
