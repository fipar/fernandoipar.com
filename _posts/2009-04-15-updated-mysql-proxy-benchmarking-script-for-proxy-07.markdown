---
layout: post
redirect_from:
  - 2009/04/15/updated-mysql-proxy-benchmarking-script-for-proxy-07/

author: Fernando Ipar
status: publish
published: true
title: Updated mysql-proxy benchmarking script (for proxy 0.7)
author_login: fernando
author_email: 
wordpress_id: 150
date: !binary |-
  MjAwOS0wNC0xNSAxOToyNzozMSAtMDMwMA==
date_gmt: !binary |-
  MjAwOS0wNC0xNSAyMToyNzozMSAtMDMwMA==
categories:
- MySQL
tags:
- MySQL
- MySQL-Proxy
- Benchmarking
comments:
- id: 1998
  author: Benchmarking Joomla | InsideMySQL
  author_email: ''
  author_url: http://www.innomysql.net/article/9977.html
  date: !binary |-
    MjAxNC0xMC0zMSAyMjowMToxNiAtMDIwMA==
  date_gmt: !binary |-
    MjAxNC0xMS0wMSAwMTowMToxNiAtMDIwMA==
  content: ! '[&#8230;] Updated mysql-proxy benchmarking script (for proxy 0.7)  My
    previous post contained a lua script for MySQL proxy&#8230; [&#8230;]'
---
<p>My previous post contained a lua script for MySQL proxy that would generate benchmarking information. However, just days (or maybe hours?) after I published it, release <a title="MySQL Proxy at Launchpad" href="https://launchpad.net/mysql-proxy">0.7</a> of mysql-proxy was published,  making my script obsolete.</p>
<p>I've fixed this (it needed just a minor tweak), so <a title="MySQL Proxy lua benchmarking script" href="http://fernandoipar.com/mysql-proxy-lua-benchmark.tar.bz2">here</a>'s a tarball with the following:</p>
<ul>
<li>trace.lua, which is the lua script to use with mysql-proxy</li>
<li>loadTrace.sh, which is a shell script that will load the output of mysql-proxy into the analysis tables. This creates the __perf database and the proper analysis table if needed. THIS LOOKS FOR trace.log IN THE CURRENT DIRECTORY. yeah yeah, it could take the name as an argument, but this is not intended as a package, it's just an example for you to follow.</li>
</ul>
<p>Remember to save the output of mysql-proxy in order to load it later with loadTrace.sh (i.e., run something like mysql-proxy --proxy-lua-script=/path/to/trace.lua &gt; trace.log and then loadTrace.sh</p>
