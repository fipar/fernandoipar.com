---
layout: post
redirect_from:
  - 2009/01/10/running-commands-from-the-shell-with-a-timeout/

author: Fernando Ipar
status: publish
published: true
title: Running commands from the shell with a timeout
author_login: fernando
author_email: 
wordpress_id: 20
date: !binary |-
  MjAwOS0wMS0xMCAwMDoyMTo0OSAtMDIwMA==
date_gmt: !binary |-
  MjAwOS0wMS0xMCAwMjoyMTo0OSAtMDIwMA==
categories:
- Programming
tags:
- Programming
- shell
- bash
comments: []
---
<p><b>EDIT: </b>This can be safely replaced by <a href="http://man7.org/linux/man-pages/man1/timeout.1.html">timeout</a></p>
<p>Sometimes, in a shell script, you need to run a command that might go on forever (or an amount of time long enough to be called forever) due to poor coding of the software and wrong external conditions (network down or overloaded, overloaded system, a changed private key, etc).</p>
<p>In order to avoid this situations, here's a small shell script that will execute any command with a guaranteed timeout:</p>
<pre>#!/bin/bash

timeout=$1
command=$2

check_pid_name()
{
        command=$1
        childpid=$2
        [ -d /proc/$childpid ] || return 1
        [ $(grep -c $command /proc/$childpid/cmdline 2&gt;/dev/null) -gt 0 ] &amp;&amp; return 0 || return 1
}

[ -z "$command" ] &amp;&amp; echo "I need a command to run"&gt;&amp;2 &amp;&amp; exit 1

[ $# -eq 2 ] &amp;&amp; shift || shift 2

$command $* &amp;
childpid=$!

sleep $timeout 

check_pid_name $command $childpid &amp;&amp; {
        kill $childpid
        sleep 0.1
        check_pid_name $command $childpid &amp;&amp; kill -9 $childpid
        echo "$command $* timed out"
}</pre>
<p>As you can see, it's quite simple. You just have to start the desired command in the background, sleep for the timeout, and then kill the process if it's still running.</p>
<p>Yes, this script has a defect, all the commands will wait $timeout to run, but that's easy to solve. I'll post an improved (but slightly more complex) version in my next post.</p>
