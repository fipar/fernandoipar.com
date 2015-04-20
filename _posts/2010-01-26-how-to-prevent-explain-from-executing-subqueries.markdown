---
layout: post
redirect_from:
  - 2010/01/26/how-to-prevent-explain-from-executing-subqueries/

status: publish
published: true
title: how to prevent explain from executing subqueries
author:
  display_name: fernando
  login: fernando
  email: 
  url: http://fernandoipar.com
author_login: fernando
author_email: 
author_url: http://fernandoipar.com
wordpress_id: 194
wordpress_url: http://fernandoipar.com/?p=194
date: !binary |-
  MjAxMC0wMS0yNiAxMDo0MzowMyAtMDIwMA==
date_gmt: !binary |-
  MjAxMC0wMS0yNiAxMzo0MzowMyAtMDIwMA==
categories:
- MySQL
tags:
- MySQL
- explain
comments:
- id: 114
  author: Kristian Nielsen
  author_email: knielsen.public@knielsen-hq.org
  author_url: http://kristiannielsen.livejournal.com/
  date: !binary |-
    MjAxMC0wMS0yNyAwNDozMDo0MiAtMDIwMA==
  date_gmt: !binary |-
    MjAxMC0wMS0yNyAwNzozMDo0MiAtMDIwMA==
  content: ! "Hm, what if there is a user-defined function (UDF) which may have side
    effects?\r\n\r\n    explain select id from projects where id = (select my_udf(max(id))
    from projects where name like 'en%');\r\n\r\nWould this EXPLAIN then actually
    call my_udf()? Seems a bit scary (but maybe one should expect that?)"
- id: 116
  author: Fernando Ipar
  author_email: 
  author_url: http://fernandoipar.com
  date: !binary |-
    MjAxMC0wMS0yNyAxNjowNToxNiAtMDIwMA==
  date_gmt: !binary |-
    MjAxMC0wMS0yNyAxOTowNToxNiAtMDIwMA==
  content: ! "Kristian: \r\n\r\nI've poked around the source before to try and figure
    things out, but code for handling explain seems to be a bit distributed on the
    code base, so it takes more than just a few minutes reading to understand it.
    \r\n\r\nSo that leaves me with my second best guess: strace. \r\n\r\nTake a look
    at this: \r\n\r\n(I'm using lookup from udf_example.so)\r\n\r\nmysql&gt; select
    lookup(url) from urls;\r\n+---------------+\r\n| lookup(url)   |\r\n+---------------+\r\n|
    69.80.205.238 | \r\n| 69.89.31.108  | \r\n| 69.80.205.238 | \r\n+---------------+\r\n3
    rows in set (0.09 sec)\r\n\r\nAnd here's the strace relevant for the lookup: \r\n\r\n[pid
    \ 8205]      0.000120 write(3, \"100127 16:40:23\\t      9 Query       select
    lookup(url) from urls\\n\"..., 65) = 65 \r\n...\r\n[pid  8205]      0.000878 open(\"/etc/resolv.conf\",
    O_RDONLY) = 37 \r\n[pid  8205]      0.000179 fstat(37, {st_mode=S_IFREG|0644,
    st_size=240, ...}) = 0 \r\n\r\n\r\nNow with explain: \r\n\r\nmysql&gt; explain
    select lookup(url) from urls;\r\n+----+-------------+-------+------+---------------+------+---------+------+------+-------+\r\n|
    id | select_type | table | type | possible_keys | key  | key_len | ref  | rows
    | Extra |\r\n+----+-------------+-------+------+---------------+------+---------+------+------+-------+\r\n|
    \ 1 | SIMPLE      | urls  | ALL  | NULL          | NULL | NULL    | NULL |    3
    |       | \r\n+----+-------------+-------+------+---------------+------+---------+------+------+-------+\r\n1
    row in set (0.00 sec)\r\n\r\n[pid  8205]      0.000137 write(3, \"100127 16:40:35\\t
    \     9 Query       explain select lookup(url) from urls\\n\"..., 73) = 73 \r\n[pid
    \ 8205]      0.000185 sched_setscheduler(8205, SCHED_OTHER, { 6 }) = -1 EINVAL
    (Invalid argument) \r\n[pid  8205]      0.000280 gettimeofday({1264617635, 884763},
    NULL) = 0 \r\n[pid  8205]      0.000091 gettimeofday({1264617635, 884852}, NULL)
    = 0 \r\n[pid  8205]      0.000266 write(29, \"\\x01\\x00\\x00\\x01\\x0a\\x18\\x00\\x00\\x02\\x03\\x64\\x65\\x66\\x00\\x00\\x00\\x02\\x69\\x64\\x00\\x0c\\x3f\\x00\\x03\\x00\\x00\\x00\\x08\\xa1\\x00\r\n\\x00\\x00\\x00\\x21\\x00\\x00\\x03\\x03\\x64\\x65\\x66\\x00\\x00\\x00\\x0b\\x73\\x65\\x6c\\x65\\x63\\x74\\x5f\\x74\\x79\\x70\\x65\\x00\\x0c\\x08\\x00\\x13\\x00\\x00\\x00\\xfd\\x01\\x00\\x1f\\x00\\\r\nx00\\x1b\\x00\\x00\\x04\\x03\\x64\\x65\\x66\\x00\\x00\\x00\\x05\\x74\\x61\\x62\\x6c\\x65\\x00\\x0c\\x08\\x00\\x40\\x00\\x00\\x00\\xfd\\x00\\x00\\x1f\\x00\\x00\\x1a\\x00\\x00\\x05\\x03\\x64\\x65\\x\r\n66\\x00\\x00\\x00\\x04\\x74\\x79\\x70\\x65\\x00\\x0c\\x08\\x00\\x0a\\x00\\x00\\x00\\xfd\\x00\\x00\\x1f\\x00\\x00\\x23\\x00\\x00\\x06\\x03\\x64\\x65\\x66\\x00\\x00\\x00\\x0d\\x70\\x6f\\x73\\x73\\x6\r\n9\\x62\\x6c\\x65\\x5f\\x6b\\x65\\x79\\x73\\x00\\x0c\\x08\\x00\\x00\\x10\\x00\\x00\\xfd\\x00\\x00\\x1f\\x00\\x00\\x19\\x00\\x00\\x07\\x03\\x64\\x65\\x66\\x00\\x00\\x00\\x03\\x6b\\x65\\x79\\x00\\x0c\r\n\\x08\\x00\\x40\\x00\\x00\\x00\\xfd\\x00\\x00\\x1f\\x00\\x00\\x1d\\x00\\x00\\x08\\x03\\x64\\x65\\x66\\x00\\x00\\x00\\x07\\x6b\\x65\\x79\\x5f\\x6c\\x65\\x6e\\x00\\x0c\\x08\\x00\\x00\\x10\\x00\\x00\\\r\nxfd\\x00\\x00\\x1f\\x00\\x00\\x19\\x00\\x00\\x09\\x03\\x64\\x65\\x66\\x00\\x00\\x00\\x03\\x72\\x65\\x66\\x00\\x0c\\x08\\x00\\x00\\x04\\x00\\x00\\xfd\\x00\\x00\\x1f\\x00\\x00\\x1a\\x00\\x00\\x0a\\x\r\n03\\x64\\x65\\x66\\x00\\x00\\x00\\x04\\x72\\x6f\\x77\\x73\\x00\\x0c\\x3f\\x00\\x0a\\x00\\x00\\x00\\x08\\xa0\\x00\\x00\\x00\\x00\\x1b\\x00\\x00\\x0b\\x03\\x64\\x65\\x66\\x00\\x00\\x00\\x05\\x45\\x7\r\n8\\x74\\x72\\x61\\x00\\x0c\\x08\\x00\\xff\\x00\\x00\\x00\\xfd\\x01\\x00\\x1f\\x00\\x00\\x05\\x00\\x00\\x0c\\xfe\\x00\\x00\\x02\\x00\\x19\\x00\\x00\\x0d\\x01\\x31\\x06\\x53\\x49\\x4d\\x50\\x4c\\x45\r\n\\x04\\x75\\x72\\x6c\\x73\\x03\\x41\\x4c\\x4c\\xfb\\xfb\\xfb\\xfb\\x01\\x33\\x00\\x05\\x00\\x00\\x0e\\xfe\\x00\\x00\\x02\\x00\"...,
    369) = 369 \r\n[pid  8205]      0.000457 gettimeofday({1264617635, 885575}, NULL)
    = 0 \r\n[pid  8205]      0.000092 sched_setscheduler(8205, SCHED_OTHER, { 8 })
    = -1 EINVAL (Invalid argument) \r\n[pid  8205]      0.000103 gettimeofday({1264617635,
    885770}, NULL) = 0 \r\n[pid  8205]      0.000084 gettimeofday({1264617635, 885853},
    NULL) = 0 \r\n[pid  8205]      0.000086 gettimeofday({1264617635, 885939}, NULL)
    = 0 \r\n[pid  8205]      0.000116 gettimeofday({1264617635, 886058}, NULL) = 0
    \r\n[pid  8205]      0.000130 write(4, \"# Time: 100127 16:40:35\\n# User@Host:
    root[root] @ localhost []\\n# Thread_id: 9  Schema: test\\n# Query_time: 0.001985
    \ Lo\r\nck_time: 0.000690  Rows_sent: 1  Rows_examined: 0  Rows_affected: 0  Rows_read:
    1\\nexplain select lookup(url) from urls;\\n\"..., 238) = 238 \r\n...\r\n\r\n\r\nTo
    discard a possible dns caching effect, I ran the original query twice on the same
    connection, and got the same strace output, so it's always going to /etc/resolv.conf\r\n\r\nStill,
    without looking at the source code, this is just anecdotical evidence from one
    example, I'll try to look more into this and perhaps make a new post on the subject
    :)"
---
<p>Here's a quick tip for using explain:</p>
<p>You may know this already, but mysql will actually execute some subqueries when you invoke explain.  Here's an example:</p>
<pre>mysql&gt; explain select id from projects where id = (select max(id) from projects where name like 'en%');
+----+-------------+----------+-------+---------------+---------+---------+-------+-------+-------------+
| id | select_type | table    | type  | possible_keys | key     | key_len | ref   | rows  | Extra       |
+----+-------------+----------+-------+---------------+---------+---------+-------+-------+-------------+
|  1 | PRIMARY     | projects | const | PRIMARY       | PRIMARY | 4       | const |     1 | Using index |
|  2 | SUBQUERY    | projects | ALL   | NULL          | NULL    | NULL    | NULL  | 67922 | Using where |
+----+-------------+----------+-------+---------------+---------+---------+-------+-------+-------------+
2 rows in set (0.11 sec)
</pre>
<p>Take a look at the execution time (I choose an intentionally poorly executing query for my little dataset).<br />
Here's explain when it's not executing:</p>
<pre>mysql&gt; explain select max(id) from projects where name like 'en%';
+----+-------------+----------+------+---------------+------+---------+------+-------+-------------+
| id | select_type | table    | type | possible_keys | key  | key_len | ref  | rows  | Extra       |
+----+-------------+----------+------+---------------+------+---------+------+-------+-------------+
|  1 | SIMPLE      | projects | ALL  | NULL          | NULL | NULL    | NULL | 69513 | Using where |
+----+-------------+----------+------+---------------+------+---------+------+-------+-------------+
1 row in set (0.00 sec)
</pre>
<p>If you want to work around this to prevent trouble on a production server (albeit, not getting the output from explain), you can do this:</p>
<pre>mysql&gt; set session max_join_size=1;
Query OK, 0 rows affected (0.00 sec)

mysql&gt; explain select id from projects where id = (select max(id) from projects where name like 'en%');
ERROR 1104 (42000): The SELECT would examine more than MAX_JOIN_SIZE rows; check your WHERE and use SET SQL_BIG_SELECTS=1 or SET SQL_MAX_JOIN_SIZE=# if the SELECT is okay
</pre>
