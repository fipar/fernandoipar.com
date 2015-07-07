---
layout: post
redirect_from:
  - 2011/03/10/piping-data-to-multiple-processes/

author: Fernando Ipar
status: publish
published: true
title: Piping data to multiple processes
author_login: fernando
author_email: 
wordpress_id: 216
date: !binary |-
  MjAxMS0wMy0xMCAwMjowMDowNiAtMDIwMA==
date_gmt: !binary |-
  MjAxMS0wMy0xMCAwNTowMDowNiAtMDIwMA==
categories:
- Programming
tags:
- Programming
- shell
comments: []
---
<p>Here's a simple shell script to stream data to multiple processes. It has many applications, but the reason I wrote it is to stream the same data to multiple netcat processes on remote machines.</p>
<p>Here's the code:</p>
<pre>
#!/bin/bash</code>

usage()
{
cat &lt;&amp;2

usage : multi-fifo target0 [target1 [target2 [...]]]

Where each targetN is a program you want to send the input multi-fifo receives

EOF

}

[ $# -eq 0 ] &amp;&amp; usage &amp;&amp; exit 1

i=0
pipeline="tee /dev/null"
while [ -n "$1" ]; do
target=$1
fifo=/tmp/multi-fifo-$$.$i
mkfifo $fifo
eval "$target &lt; $fifo" &amp;
pipeline="$pipeline | tee $fifo"
i=$((i+1))
shift
done

eval "cat | $pipeline"

j=0
while [ $j -lt $i ]; do
rm -f /tmp/multi-fifo-$$.$j
j=$((j+1))
done
</pre>
<p>Usage is pretty straightforward:</p>
<p><code><br />
tar cjvf - . | ./multi-fifo "nc host1 9999" "nc host2 9999" &gt;/dev/null<br />
</code></p>
<p>This way, the script will create a fifo for each netcat client, then send it's stdin to a pipeline that writes to both fifos using tee. Useful for example if you want to stream a backup to multiple destination servers for cloning.</p>
