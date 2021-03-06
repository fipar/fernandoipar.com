---
layout: post
redirect_from:
  - 2014/10/31/benchmarking-joomla/

author: Fernando Ipar
status: publish
published: true
title: Benchmarking Joomla
author_login: fernando
author_email: 
wordpress_id: 331
date: !binary |-
  MjAxNC0xMC0zMSAxNDoxNjo0NSAtMDIwMA==
date_gmt: !binary |-
  MjAxNC0xMC0zMSAxNzoxNjo0NSAtMDIwMA==
categories:
- MySQL
tags:
- Benchmarking
comments: []
---
<p><a href="https://mariadb.com/blog/why-i-moved-my-joomla-website-mariadb">This post</a> recently caught my attention on Planet MySQL.</p>
<p>If you haven't read it yet, I suggest that you go and do so, and also read the comments.</p>
<p>I think Matthew's request for the queries so that others can run comparative benchmarks is very interesting, and while I don't have access to the queries used to produce those results, I am making some queries of my own available for others to test. You can get your own queries if you want, just enable slow query logging with long_query_time set to 0 on the database for a Joomla instalation (being unexperienced in Joomla I used <a href="https://github.com/joomlatools/joomla-vagrant">this project</a>, which is great) and then run these steps on the resulting slow query log file:</p>
<p><code><br />
csplit -z joomlatools-slow.log '/^#/' '{*}'<br />
for f in x*; do grep -i select $f &gt;/dev/null || rm -f $f; done<br />
for f in x*; do grep -v '^SET' $f &gt; $$ &amp;&amp; mv -f $$ $f; done<br />
for f in x*; do grep -v '^#' $f &gt; $$ &amp;&amp; mv -f $$ $f; done<br />
for f in x*; do tr '\n' ' ' &lt;$f&gt;$$ &amp;&amp; echo &gt;&gt; $$ &amp;&amp; mv -f $$ $f; done<br />
cat x* &gt; queries.txt<br />
</code></p>
<p>That should give you a file similar to <a href="https://gist.github.com/fipar/3bf5b48685e89e7199f3">this one.</a><br />
The EXPLAIN statements are not mine, everything you see on that file was generated by me visiting a sample Joomla site. </p>
<p>I do not have the time to run a big benchmark now, but a mysqlslap using the exact same settings as those from the original post, and my list of queries, seems simple enough. </p>
<p>The vagrant box for Joomla has MySQL 5.5 so I used that for my tests. Here are the results: </p>
<p>MySQL 5.5 with the default configuration (default as installed by the debian package):<br />
<code><br />
root@joomlatools:~# mysqlslap --concurrency=100 --iterations=10 --query=queries.txt --create-schema=myuser_joomla -umyuser_joom -p7654e684c4d14ab544261568101cc9c4<br />
Benchmark<br />
	Average number of seconds to run all queries: 1.911 seconds<br />
	Minimum number of seconds to run all queries: 1.786 seconds<br />
	Maximum number of seconds to run all queries: 2.202 seconds<br />
	Number of clients running queries: 100<br />
	Average number of queries per client: 265<br />
</code></p>
<p>MariaDB 10.1, with the config as installed by the MariaDB package:<br />
<code><br />
root@joomlatools:~# mysqlslap --concurrency=100 --iterations=10 --query=queries.txt --create-schema=myuser_joomla -umyuser_joom -p7654e684c4d14ab544261568101cc9c4<br />
Benchmark<br />
	Average number of seconds to run all queries: 1.743 seconds<br />
	Minimum number of seconds to run all queries: 1.694 seconds<br />
	Maximum number of seconds to run all queries: 1.847 seconds<br />
	Number of clients running queries: 100<br />
	Average number of queries per client: 265<br />
</code></p>
<p>MySQL 5.5 with MariaDB's configuration (minus incompatible options)<br />
<code><br />
root@joomlatools:~# mysqlslap --concurrency=100 --iterations=10 --query=queries.txt --create-schema=myuser_joomla -umyuser_joom -p7654e684c4d14ab544261568101cc9c4<br />
Benchmark<br />
	Average number of seconds to run all queries: 1.803 seconds<br />
	Minimum number of seconds to run all queries: 1.548 seconds<br />
	Maximum number of seconds to run all queries: 2.398 seconds<br />
	Number of clients running queries: 100<br />
	Average number of queries per client: 265<br />
</code></p>
<p>Interesting. MariaDB still performs better than 5.5, but performance improved for this old version anyway.<br />
I took a quick look at the configuration file and among the changes, use of the query cache caught my attention. I think it's strange to have it enabled by default since it can cause so much contention problems at heavy concurrency workloads, but I'm not familiar with the MariaDB improvements and perhaps the query cache implementation on it is better at scaling. </p>
<p>Just to check, I tried MariaDB 10.1 with the query cache disabled:<br />
<code><br />
mariadb 10.1 without the query cache:<br />
root@joomlatools:~# mysqlslap --concurrency=100 --iterations=10 --query=queries.txt --create-schema=myuser_joomla -umyuser_joom -p7654e684c4d14ab544261568101cc9c4<br />
Benchmark<br />
	Average number of seconds to run all queries: 4.113 seconds<br />
	Minimum number of seconds to run all queries: 3.735 seconds<br />
	Maximum number of seconds to run all queries: 5.446 seconds<br />
	Number of clients running queries: 100<br />
	Average number of queries per client: 265<br />
</code></p>
<p>Interesting. It seems without the query cache, MariaDB 10.1 performs worse than MySQL 5.5. Could be a concurrency thing though, as I know more recent MySQL versions (and I assume the same is true for MariaDB) have worked on optimising for high concurrency use cases, at the expense of low concurrency or single threaded workloads. And while the mysqlslap invocation runs 100 threads, don't be fooled. I ran this on a VM on a notebook with 2 cores with hyperthreading. It's great as a workstation but it wouldn't cut it as a server :)</p>
<p>I'd like to see what experience others have. You know exactly the VM and workload I used for this tests, so let's see what numbers you get. </p>
