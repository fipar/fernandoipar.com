---
layout: post
redirect_from:
  - 2015/03/11/ongoing-mysql-myths/

status: publish
published: true
title: Ongoing MySQL myths
author:
  display_name: fernando
  login: fernando
  email: 
  url: http://fernandoipar.com
author_login: fernando
author_email: 
author_url: http://fernandoipar.com
wordpress_id: 337
wordpress_url: http://fernandoipar.com/?p=337
date: !binary |-
  MjAxNS0wMy0xMSAwOTo0NTowNyAtMDMwMA==
date_gmt: !binary |-
  MjAxNS0wMy0xMSAxMjo0NTowNyAtMDMwMA==
categories:
- MySQL
tags:
- MySQL
comments:
- id: 3123
  author: Roland Bouman
  author_email: roland.bouman@gmail.com
  author_url: http://rpbouman.blogspot.com
  date: !binary |-
    MjAxNS0wMy0xMSAxMDo1MjoxNiAtMDMwMA==
  date_gmt: !binary |-
    MjAxNS0wMy0xMSAxMzo1MjoxNiAtMDMwMA==
  content: ! "Nice write up! Thanks for the shout-out :)\r\n\r\nRoland."
- id: 3126
  author: Jaime Crespo
  author_email: jynus@jynus.com
  author_url: http://dbahire.com
  date: !binary |-
    MjAxNS0wMy0xMSAxMTo0NzozMyAtMDMwMA==
  date_gmt: !binary |-
    MjAxNS0wMy0xMSAxNDo0NzozMyAtMDMwMA==
  content: ! "Thank you for this.\r\n\r\nAs an addition: since 5.6, GA more than 2
    years ago, Oracle ships its MySQL packages with strict SQL defaults on the config
    file (people may not have noticed because of Linux distribution defaults or them
    using ancients versions) and compiled in defaults in 5.7.\r\n\r\nThe only reason
    the change is taking so long is due to backwards compatibility with old applications.\r\n\r\nOther
    myth that people continue repeating over and over without really understanding
    the internals is that data is not really safe in MySQL: By default, MySQL is fully
    ACID, synchronous to disk, data is written twice to secondary storage to avoid
    partial page writes in case the system crashes, and mantains a checksum for every
    16K bytes of information, shutting down automatically if it detects physical corruption.
    Despite that, it still beats some of the other systems in throughput and latency.\r\n\r\nI
    would love if some people stopped attacking MySQL and instead praised their favourite
    DB system. Or better, stopped having a favourite system and praised the strong
    points and hated the weak ones of every option. Like programming languages, every
    option has its cons and pros."
- id: 3127
  author: Mark Grennan
  author_email: mark@grennan.com
  author_url: http://www.mysqlfanboy.com
  date: !binary |-
    MjAxNS0wMy0xMSAxMjozMjoxNyAtMDMwMA==
  date_gmt: !binary |-
    MjAxNS0wMy0xMSAxNTozMjoxNyAtMDMwMA==
  content: As a "Fan Boy" I have to say, Thanks for a good post.    I'll go out on
    a limb and say... As a push by every App Dev to become a rock star and write their
    own data store,  MySQL has fallen into the "Old School" bucket with Oracle.   DBA's
    are being ask to for the "New Stuff" without being ask for their expertise.  Things
    like the memcache interface to MySQL are simply ignored by developers.
- id: 3130
  author: Justin Swanhart
  author_email: justin.swanhart@percona.com
  author_url: ''
  date: !binary |-
    MjAxNS0wMy0xMSAxNDowMzowNCAtMDMwMA==
  date_gmt: !binary |-
    MjAxNS0wMy0xMSAxNzowMzowNCAtMDMwMA==
  content: ! "pgsql has bitmap join optimization, in that it can scan more than one
    b-tree, create a bitmap and logically combine them.  This is faster than \"merge
    intersection\" that MySQL has.  pgsql doesn't have real bitmap indexes to the
    best of my knowledge, unless perhaps they were added in the latest release and
    I missed them.\r\n\r\nAnd if you want real bitmap indexes for MySQL, you can always
    use my FastBit_UDF.  It is a little clunky, but very very fast."
- id: 3131
  author: fernando
  author_email: 
  author_url: http://fernandoipar.com
  date: !binary |-
    MjAxNS0wMy0xMSAxNDoyNzoyMyAtMDMwMA==
  date_gmt: !binary |-
    MjAxNS0wMy0xMSAxNzoyNzoyMyAtMDMwMA==
  content: Justin, I was thinking of the Bizgres patches that did/do provide on-disk
    bitmap indexes, but it seems it never made it into a stable version (I'll admit
    I haven't seriously played with PostgreSQL in a while...)
---
<p>There's an interesting post over at <a href="http://developer.olery.com/blog/goodbye-mongodb-hello-postgresql/">Olery</a>'s blog about a successful migration story from using MySQL/MongoDB to PostgreSQL as the persistence layer for applications that, however, lists a couple of cons of using MySQL that I personally think are no longer valid complaints (or at least not as big as they used to be). I did not find a way to reply to the post so, hopefully for the benefit of people who are new to MySQL and may hear the same recycled myths, I am doing so here.</p>
<p>Now I am aware that PostgreSQL has feature MySQL lacks (bitmap indexes are a personal favourite), but still, it is frustrating that instead of using actual arguments consisting of actual use cases in which one database is better than another, these old stories get recycled. I think it's a disservice to both MySQL and PostgreSQL.</p>
<p>The first paragraph that caught my attention is:</p>
<blockquote><p>MySQL was the first candidate as we were already using it for some small chunks of critical data. MySQL however is not without its problems. For example, when defining a field as <code>int(11)</code> you can just happily insert textual data and MySQL will <em>try</em> to convert it.</p></blockquote>
<p>The <a href="http://dev.mysql.com/doc/refman/5.6/en/sql-mode.html">sql_mode</a> variable is available since at least <a href="http://dev.mysql.com/doc/refman/4.1/en/sql-mode.html">4.0</a> (GA in 2004 if I'm not mistaken, which is over a decade ago) and while yes, the default value is permissive, and any evil programmer will be able to change the value for the variable (Roland made <a href="http://rpbouman.blogspot.com/2009/01/mysqls-sqlmode-my-suggestions.html">very good suggestions</a> about this some time ago), in my experience it has provided people with enough protection against this kind of data integrity problems.</p>
<p>And here's the second one:</p>
<blockquote><p>Another problem with MySQL is that any table modification (e.g. adding a column) will result in the table being locked for <em>both</em> reading and writing. This means that <em>any</em> operation using such a table will have to wait until the modification has completed.</p></blockquote>
<p>Innodb supports <a href="http://dev.mysql.com/doc/refman/5.6/en/innodb-online-ddl.html">some online DDL</a> operations starting with MySQL 5.6 (GA in February 2013, over two years ago) and of course, there's always <a href="http://www.percona.com/doc/percona-toolkit/2.2/pt-online-schema-change.html">pt-online-schema-change</a>, which can usually help in the cases were online DDL is still not supported, or when the impact on replica lag makes that approach unfeasible.</p>
<p>To wrap up, my message is that it's a big world out there, with different technologies that all have cons and pros. You need to make your due diligence and investigate the biggest of each when evaluating one technology, or comparing a few of them, to make sure you pick the best tool for the job (and for your context, because tools are not context-free. A kitchen knife is not a very good screwdriver, but a few times in my life I found myself in the situation of using it, and with success!), but most importantly, please base your decision on actually experience-verified facts, and not on things you've heard others say, or that perhaps you experienced a long time ago.</p>
<p>&nbsp;</p>
