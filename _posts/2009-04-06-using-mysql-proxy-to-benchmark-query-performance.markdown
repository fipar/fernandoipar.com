---
layout: post
redirect_from:
  - 2009/04/06/using-mysql-proxy-to-benchmark-query-performance/

status: publish
published: true
title: Using MySQL Proxy to benchmark query performance
author:
  display_name: fernando
  login: fernando
  email: 
  url: http://fernandoipar.com
author_login: fernando
author_email: 
author_url: http://fernandoipar.com
wordpress_id: 139
wordpress_url: http://fernandoipar.com/?p=139
date: !binary |-
  MjAwOS0wNC0wNiAwOToyMDoyOCAtMDMwMA==
date_gmt: !binary |-
  MjAwOS0wNC0wNiAxMToyMDoyOCAtMDMwMA==
categories:
- MySQL
tags:
- MySQL
- performance
- MySQL-Proxy
- Benchmarking
comments:
- id: 37
  author: php trivandrum
  author_email: jijutm@gmail.com
  author_url: http://www.php-trivandrum.org
  date: !binary |-
    MjAwOS0wNC0xMCAyMjo0Mjo0MiAtMDMwMA==
  date_gmt: !binary |-
    MjAwOS0wNC0xMSAwMDo0Mjo0MiAtMDMwMA==
  content: ! "Should have seen this a fortnight back. And would not have gone to put
    down code for Open PHP MyProfiler http://www.php-trivandrum.org/open-php-myprofiler.\r\n\r\nOpen
    PHP MyProfiler is just a trial to run query profiling on a php-mysql application,
    without changing the architecture too much. The profiler is open and downloadable,
    though the analzer, is just a mockup or a bare one and not yet ripe to be opened
    up. Any one who needs to do the analysis could make use of the same by downloading
    the profiler, and implementing with their code. The profiler would create logs
    depending on hostname and date. upload the profile logs to our Profile Sampler
    and you should be able to see the full profile of your application."
- id: 38
  author: php trivandrum
  author_email: jijutm@gmail.com
  author_url: http://www.php-trivandrum.org
  date: !binary |-
    MjAwOS0wNC0xMCAyMzoxNzoxNyAtMDMwMA==
  date_gmt: !binary |-
    MjAwOS0wNC0xMSAwMToxNzoxNyAtMDMwMA==
  content: ! "I did try this out.. first thing, in your lua script, should it not
    be \"in\", instead of \"inj\" as parameter for read_query_result()?\r\n\r\nAlso
    to achieve this, you dont need the read_query and the global query, since in.query
    will give the original query as read here: http://dev.mysql.com/doc/mysql-ha-scalability/en/mysql-proxy-scripting-read-query-result.html"
- id: 39
  author: php trivandrum
  author_email: jijutm@gmail.com
  author_url: http://www.php-trivandrum.org
  date: !binary |-
    MjAwOS0wNC0xMCAyMzoyNDowMiAtMDMwMA==
  date_gmt: !binary |-
    MjAwOS0wNC0xMSAwMToyNDowMiAtMDMwMA==
  content: Sorry, I was wrong in the second part. To trigger read_query_result, the
    first injection is needed
- id: 40
  author: fernando
  author_email: mail@fernandoipar.com
  author_url: http://fernandoipar.com
  date: !binary |-
    MjAwOS0wNC0xMSAwMDo0OTowNiAtMDMwMA==
  date_gmt: !binary |-
    MjAwOS0wNC0xMSAwMjo0OTowNiAtMDMwMA==
  content: ! "Hi, \r\n\r\nThis was my first lua script, so some parts might have been
    wrong. Still, I think the first injection was needed, and your third message seems
    to confirm this so it gives me some confidence ;)\r\n\r\nRegarding your second
    message, it's possible. There was a new release of mysql-proxy just after I published
    this, and apparently, some changes rendered my script obsolete. I've joined the
    mysql-proxy team on launchpad to try to keep a closer eye on the project and I
    promise to make an updated post on this blog with a new script that works. \r\n\r\nIn
    fact, if it's approved by the team, I'll publish in the cookbook where it'll be
    much more useful. \r\n\r\nI think your project is quite interesting and might
    be in fact easier to implement for people working on the LAMP stack and on shared
    hosting environments (where installing the proxy is totally out of the question),
    so please keep up the good work!"
- id: 41
  author: php trivandrum
  author_email: jijutm@gmail.com
  author_url: http://www.php-trivandrum.org
  date: !binary |-
    MjAwOS0wNC0xNCAwMDoyNTowNiAtMDMwMA==
  date_gmt: !binary |-
    MjAwOS0wNC0xNCAwMjoyNTowNiAtMDMwMA==
  content: ! "hi fernando, \r\n\r\nI never thought about the shared hosting part,
    thanks for boosting my morale, I would be quite happy if you would comment on
    my relevant page\r\n\r\nhttp://www.php-trivandrum.org/open-php-myprofiler"
---
<p>By transparently sitting between client and server on each request, MySQL Proxy offers many possibilities for query manipulation.</p>
<p>Many are explored in the <a title="MySQL Proxy Cookbook" href="forge.mysql.com/wiki/MySQL_Proxy_Cookbook">cookbook</a>, and they even include a histogram recipe. Still, I wanted to learn more about the proxy while working on a script that would let me get some stats on the queries executed against a server (or group of servers).</p>
<p>First things first, get a brief glimpse of the <a title="The lua programming language" href="www.lua.org">lua programming language</a> since that's what the proxy's scripts are written in. Alternatively, you can jump straight into the sample scripts, extrapolate what you don't understand of the syntax by making paralelizations against other known scripting languages and make the best of it. That's what I've been doing so far :)</p>
<p>We'll, now on to it.</p>
<p>Here's my super simple proxy script. It consists of a global variable and a line that spits out all the vars I want, separated by '||||'. I choose that separator since it's unlikely to happen in a real query and hence it won't cause me to loose much data while doing load data infile later. That's my scientific approach, and considering this aren't medical records, it's good enough for me.</p>
<p>I can get away with using a global var because the proxy fires up a new instance of the lua script for every new client connection. At least that's what I've been able to find out so far, and my empirical data has confirmed this. If source code inspection later rejects this finding, I'll have to find a better (probably more complex) way to achieve the same goal.</p>
<pre>query = ""

function read_query( packet )
	if packet:byte() == proxy.COM_QUERY then
		query = packet:sub(2)
		proxy.queries:append(1, packet )
		return proxy.PROXY_SEND_QUERY
	end
end

function read_query_result(inj)
        print(os.date('%Y-%m-%d %H:%M:%S') .. "||||" ..  query .. "||||" .. (in.query_time / 1000) .. "||||" .. (in.response_time / 1000))
end</pre>
<p>That simple script saves the query into the global variable, from the read_query hook function, and prints the results in the read_query_results function. Notice how this hooks provide for much more possibilities if you're a skillful hacker and an evil one too :) (i.e., man in the middle type of things, there are a few query modification examples in the cookbook)</p>
<p>Ok, so with this part covered, we need to run the proxy, and then run some queries against it.</p>
<p>Both things are easy:</p>
<pre>$ mysql-proxy --proxy-lua-script trace.lua &amp;&gt; trace.log</pre>
<p>and something like</p>
<pre>$ mysql -someuser -psomepassword -proxyhost -P4040</pre>
<p>Notice that 4040 is the default port for the proxy, but you could change it into 3306, and move mysql into another port.</p>
<p>This generates a trace.log file that looks like this:</p>
<pre>2009-04-06 07:13:03||||select count(*) from City||||0.381||||0.404
2009-04-06 07:13:06||||desc City||||2.08||||2.18
2009-04-06 07:13:20||||select * from City where Population between 200 and 2000||||169.122||||194.083
2009-04-06 07:13:22||||select * from City where Population between 200 and 2000||||0.408||||9.16
2009-04-06 07:13:23||||select * from City where Population between 200 and 2000||||0.664||||8.455</pre>
<p>We need to load that into mysql.</p>
<p>I created a script just for that purpose:</p>
<pre>#!/bin/bash

tcp trace.log /tmp

cd=$(date "+%Y%m%d%H%M%S")

echo "You'll be asked for MySQL root user's password"

mysql -root -p &lt;&lt;EOSCR
set @@sql_mode=ANSI;
create database if not exists "__perf";
use __perf;

create table if not exists "analysis_results_${cd}"(
        id int unsigned not null auto_increment,
        ts datetime,
        query char(200),
        query_time float,
        response_time float,
        primary key ( id ),
        key ( ts ),
        key ( query(100) ),
        fulltext ( query ),
        key ( query_time ),
        key ( response_time )
) Engine=MyISAM ROW_FORMAT=Fixed;

load data infile '/tmp/trace.log' into table "analysis_results_${cd}" fields terminated by '||||' (ts,query,query_time,response_time);

EOSCR

resultMySQL=$?

rm -f /tmp/trace.log

[ $resultMySQL -eq 0 ] &amp;&amp; echo "Data imported OK"&gt;&amp;2 || echo "Error while importing data, please refer to the output of MySQL"&gt;&amp;2</pre>
<p>So, once you've run quite a few queries against the proxy, you could do something like this:</p>
<pre>$ ./loadTrace.sh
You'll be asked for MySQL root user's password
Enter password:
Data imported OK</pre>
<p>And then run some queries against the analysis tables.</p>
<p>Here are some ideas:</p>
<p>Top 10 queries that took more time to process:</p>
<pre>mysql&gt; select * from analysis_results_20090406073836 order by query_time desc limit 10;</pre>
<p>Top 10 queries that took more time to return to the client:</p>
<pre>mysql&gt; select * from analysis_results_20090406073836 order by response_time desc limit 10;</pre>
<p>Top 10 queries, ordered by their text, and then the time it took them to get back. You can infer, by the query issue time, if the query cache was in use, and then, if it was useful. You'll be surprised that for large datasets, while the load is taken off the server by using the cache, the client doesn't perceive such a big improvement because it still takes a lot of time for the resultset to go back. Therefore he/she will still complain. Sometimes a lot of effort is put into optimizing server performance, and the way this server is accessed is totally neglected!</p>
<pre>mysql&gt; select * from analysis_results_20090406073836 order by query, response_time desc limit 10;</pre>
<p>Well, there you go. Have fun and find out what kind of usage your application is giving to your server.</p>
