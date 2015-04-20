---
layout: post
redirect_from:
  - 2009/02/04/generating-random-salts-from-bash/

status: publish
published: true
title: Generating random salts from bash
author:
  display_name: fernando
  login: fernando
  email: 
  url: http://fernandoipar.com
author_login: fernando
author_email: 
author_url: http://fernandoipar.com
wordpress_id: 86
wordpress_url: http://fernandoipar.com/?p=86
date: !binary |-
  MjAwOS0wMi0wNCAyMToxOTo1NyAtMDIwMA==
date_gmt: !binary |-
  MjAwOS0wMi0wNCAyMzoxOTo1NyAtMDIwMA==
categories:
- Programming
tags:
- Programming
- bash
- security
- perl
comments: []
---
<p>From the 'just because it can be done' column, here comes a handy shell script to generate random <a title="Cryptographic Salt (Wikipedia)" href="http://en.wikipedia.org/wiki/Salt_(cryptography)">salts</a>.</p>
<p>So, without further ado,  here it goes:</p>
<pre>#!/bin/bash 

[ $# -eq 0 ] &amp;&amp; {
        echo "usage: salt &lt;length&gt;"&gt;&amp;2
        exit
}
strings &lt;/dev/urandom | while read line; do
        echo $line | tr '\n\t ' $RANDOM:0:1 &gt;&gt; /tmp/.salt.$$
        salt=$(cat /tmp/.salt.$$)
        [ ${#salt} -ge $1 ] &amp;&amp; salt=${salt:0:$1} &amp;&amp; echo $salt &amp;&amp; break
done
rm -f /tmp/.salt.$$</pre>
<p>I had to use $1 and not a var, and echo the salt right from inside the while, because the '|' creates another shell, so I can't pass variables to or from the while in this case.</p>
<p>If you want to keep things simple, you can go perl and just do</p>
<pre>
#!/usr/bin/perl
use Crypt::Salt;
my $length = shift;
print salt($length);
</pre>
<p>But if you ever find yourself in a server with no cpan, the first option might prove useful :)</p>
