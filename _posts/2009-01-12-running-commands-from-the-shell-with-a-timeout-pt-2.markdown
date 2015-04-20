---
layout: post
redirect_from:
  - 2009/01/12/running-commands-from-the-shell-with-a-timeout-pt-2/

status: publish
published: true
title: Running commands from the shell with a timeout (pt 2)
author:
  display_name: fernando
  login: fernando
  email: 
  url: http://fernandoipar.com
author_login: fernando
author_email: 
author_url: http://fernandoipar.com
wordpress_id: 29
wordpress_url: http://fernandoipar.com/?p=29
date: !binary |-
  MjAwOS0wMS0xMiAyMDo0NTozMyAtMDIwMA==
date_gmt: !binary |-
  MjAwOS0wMS0xMiAyMjo0NTozMyAtMDIwMA==
categories:
- Programming
tags:
- Programming
- shell
- bash
comments:
- id: 2
  author: Pádraig Brady
  author_email: P@draigBrady.com
  author_url: http://www.pixelbeat.org/
  date: !binary |-
    MjAwOS0wMS0xMyAwODowMjo0NCAtMDIwMA==
  date_gmt: !binary |-
    MjAwOS0wMS0xMyAxMDowMjo0NCAtMDIwMA==
  content: ! "Seems a bit complicated. Have a look at:\r\nhttp://www.pixelbeat.org/scripts/timeout\r\n\r\nAlso
    note that there will be a timeout command included in newer versions of coreutils."
- id: 3
  author: fernando
  author_email: mail@fernandoipar.com
  author_url: http://fernandoipar.com
  date: !binary |-
    MjAwOS0wMS0xMyAxMjowODozNyAtMDIwMA==
  date_gmt: !binary |-
    MjAwOS0wMS0xMyAxNDowODozNyAtMDIwMA==
  content: ! "Thanks for the reply. I had two similar replies in reddit, I guess I
    was really out of touch. \r\n\r\nI've just installed timeout through apt-get and
    it does just what my script from Highbase does, though I'll wait until it's included
    in coreutils to remove my self made version. \r\n\r\nI tried to figure out when
    timeout is coming with coreutils, and it seems you're the contributor :) Any ideas
    for a date?"
---
<p>Here's an improved version of the safecmd script. This one doesn't always wait for $timeout seconds even if the command you're running exits successfully. In that case, it kills the monitor script and ends at the proper time. </p>
<p>Here's the code: </p>
<pre>
#!/bin/bash 

timeout=$1
command=$2
shift 2

check_pid_name()
{
        command=$1
        childpid=$2
        [ -d /proc/$childpid ] || return 1
        [ $(grep -c $command /proc/$childpid/cmdline 2>/dev/null) -gt 0 ] && return 0 || return 1
}

[ -z "$command" ] && echo "Usage: safecmd <timeout> <command> [args]">&2 && exit 1

safe_run()
{
        command=$1
        shift
        $command $* 
        kill $$
}

safe_run $command $* &
childpid=$!
sleep $timeout 

check_pid_name $command $childpid && {
        kill $childpid
        sleep 0.1
        check_pid_name $command $childpid && kill -9 $childpid
        echo "$command $* timed out"
}
</pre>
<p>You can try it how like this, for example: </p>
<pre>
./safecmd 2 sleep 4
</pre>
<p>which will output</p>
<pre>
./safecmd.1: line 34: 11739 Terminated               safe_run $command $*
sleep 4 timed out
</pre>
<pre>
or ./safecmd 2 sleep 1
</pre>
<p>which will output</p>
<pre>
Terminated
</pre>
<p>Indicating that sleep 1 finished successfully</p>
<p>This is still missing something. You can't test for the success of the command you pass safecmd, if it exits successfully you'll see the same return code than if it would have exited with an error. </p>
<p>In order to improve this, as far as I can tell, you have to use two scripts, one just as the wrapper (to use one separate script instead of the function safe_run()). That's the way this is handled in <a href="http://highbase.seriema-systems.com">Highbase</a>, and I'll post a full example in my next post on this subject. </p>
