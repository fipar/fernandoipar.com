---
layout: post
redirect_from:
  - 2014/02/09/recovering-mysql-access/

status: publish
published: true
title: Recovering MySQL access
author:
  display_name: fernando
  login: fernando
  email: 
  url: http://fernandoipar.com
author_login: fernando
author_email: 
author_url: http://fernandoipar.com
wordpress_id: 310
wordpress_url: http://fernandoipar.com/?p=310
date: !binary |-
  MjAxNC0wMi0wOSAxMTozMzo0NyAtMDIwMA==
date_gmt: !binary |-
  MjAxNC0wMi0wOSAxNDozMzo0NyAtMDIwMA==
categories:
- MySQL
tags:
- security
- MySQL
comments:
- id: 259
  author: MySQL忘记root密码 | DBA的罗浮宫
  author_email: ''
  author_url: http://mdba.cn/?p=424
  date: !binary |-
    MjAxNC0wMi0xNiAxMDo1OTozMiAtMDIwMA==
  date_gmt: !binary |-
    MjAxNC0wMi0xNiAxMzo1OTozMiAtMDIwMA==
  content: ! '[&#8230;] 3、不重启的mysqld的方法 还有人问我，我忘记了root密码，但又不能重启mysqld，如何重置root密码？
    在没有看到Fernando的博客Recovering MySQL Access之前，我几乎认为没法完成这样的工作，Fernando的方法让人简直令人惊喜，让我学到了。
    从上面可以看到两种方法可以看到，mysql的用户账号和密码存储在mysql.user表中。一只以来，我一直处于一个误区中，mysql数据中的表都是innodb存储引擎的，而实际上绝大多数表示是以myisam作为存储引擎。
    mysql&gt; select count(*),ENGINE from tables where TABLE_SCHEMA =&#8217;mysql&#8217;
    group by ENGINE; [&#8230;]'
---
<p>Ever found yourself working on a MySQL server where root's password is unavailable? It has happened to me a few times, always because the person who set up the DB left the place long ago, and this information was not documented anywhere.</p>
<p>If you have root access to the OS, MySQL lets you restart the server bypassing access checks, using the <a href="http://dev.mysql.com/doc/refman/5.6/en/server-options.html#option_mysqld_skip-grant-tables">skip-grant-tables</a> option, which requires a service restart.</p>
<p>However, if you need to regain root access and want to minimize service impact, you can take advantage of the way the server responds to <a href="http://dev.mysql.com/doc/refman/5.6/en/server-signal-response.html">SIGHUP</a> signals and the fact that access credentials are stored on a MyISAM table.</p>
<p>MySQL uses a few tables to store credentials and privileges for users (you can find more about this <a href="https://dev.mysql.com/doc/refman/5.5/en/grant-table-structure.html">here</a>), but for this procedure, we only need to work with the mysql.user table.</p>
<p>Specifically, we will work with the columns 'user', 'host' and 'password' from this table.</p>
<p>Here's an example of how this can look on a server:</p>
<pre>mysql&gt; select user,host,password from mysql.user;
+-----------+-----------+-------------------------------------------+
| user      | host      | password                                  |
+-----------+-----------+-------------------------------------------+
| root      | localhost | *1BD9C328233CF457571A4BB5DB8D32892AB8EDBF |
| root      | mysql     | *1BD9C328233CF457571A4BB5DB8D32892AB8EDBF |
| root      | 127.0.0.1 | *1BD9C328233CF457571A4BB5DB8D32892AB8EDBF |
| root      | ::1       | *1BD9C328233CF457571A4BB5DB8D32892AB8EDBF |
|           | localhost |                                           |
|           | mysql     |                                           |
| dba       | %         | *4FC8D8270BEC4364C78799065996F5306139B412 |
| readwrite | localhost | *202273E75BD11D06FBE2F057BFA1B1BB2B26549C |
| readonly  | localhost | *FC69E042CE30D92E2952335F690CF2345C812E36 |
+-----------+-----------+-------------------------------------------+
9 rows in set (0.00 sec)</pre>
<p>To start, we'll need to make a copy of this table to a database where we can change it. On this example server, this means the 'test' schema, as the 'readwrite' user has write privileges on it. Even if root's password was lost, you can typically get a less privileged MySQL account by checking the applications that connects to this database. If for some reason this is not the case, you can achieve the same results by copying this table to another server, and copying it back after the necessary changes have been made.</p>
<p>The following command happen on the datadir:</p>
<pre>[root@mysql mysql]# cp mysql/user.* test/; chown mysql.mysql test/user.*</pre>
<p><strong>Please don't overwrite an existing table</strong> when doing this! Rename the copied files as needed instead ...</p>
<p>Now you should be able to access (and write) to this table:</p>
<pre>[root@mysql mysql]# mysql -ureadwrite -p test
Enter password: 
Reading table information for completion of table and column names
You can turn off this feature to get a quicker startup with -A

Welcome to the MySQL monitor.  Commands end with ; or \g.
Your MySQL connection id is 34
Server version: 5.6.16 MySQL Community Server (GPL)

Copyright (c) 2000, 2014, Oracle and/or its affiliates. All rights reserved.

Oracle is a registered trademark of Oracle Corporation and/or its
affiliates. Other names may be trademarks of their respective
owners.

Type 'help;' or '\h' for help. Type '\c' to clear the current input statement.

mysql&gt; select user,host,password from user;
+-----------+-----------+-------------------------------------------+
| user      | host      | password                                  |
+-----------+-----------+-------------------------------------------+
| root      | localhost | *1BD9C328233CF457571A4BB5DB8D32892AB8EDBF |
| root      | mysql     | *1BD9C328233CF457571A4BB5DB8D32892AB8EDBF |
| root      | 127.0.0.1 | *1BD9C328233CF457571A4BB5DB8D32892AB8EDBF |
| root      | ::1       | *1BD9C328233CF457571A4BB5DB8D32892AB8EDBF |
|           | localhost |                                           |
|           | mysql     |                                           |
| dba       | %         | *4FC8D8270BEC4364C78799065996F5306139B412 |
| readonly  | %         | *FC69E042CE30D92E2952335F690CF2345C812E36 |
| readwrite | %         | *202273E75BD11D06FBE2F057BFA1B1BB2B26549C |
+-----------+-----------+-------------------------------------------+
9 rows in set (0.00 sec)</pre>
<p>By now you've probably figured out what I'll do: update test.user, changing the password column for user 'root' and host 'localhost' to the result of running the PASSWORD() function with some string of my choice, then copying this table back, and then sending SIGHUP to the server.</p>
<p>A couple of caveats:</p>
<ul>
<li>Either make a copy of the original table file, (and?) or write down the original hash for root (the one you will replace)</li>
<li>Even if nobody on the customer's current team knows how to get you MySQL's root password, that does not mean they don't have some old app someone has forgotten about that uses the root account to connect. If this is the case, access will break for this app. You can follow the same steps outlined here, but instead of permanently changing root's password, use your regained access to create a new super user account, and then replace root's hash with the one you saved (and flush privileges!)</li>
</ul>
<p>For completion, here's the rest of the process:</p>
<pre>mysql&gt; update test.user set password=password('newpass but this is insecure so dont use') where user = 'root' and host = 'localhost';
Query OK, 1 row affected (0.00 sec)
Rows matched: 1  Changed: 1  Warnings: 0

mysql&gt; select user,host,password from test.user where user='root';
+------+-----------+-------------------------------------------+
| user | host      | password                                  |
+------+-----------+-------------------------------------------+
| root | localhost | *0A131BF1166FB756A61317A40F272D6FFDD281E9 |
| root | mysql     | *1BD9C328233CF457571A4BB5DB8D32892AB8EDBF |
| root | 127.0.0.1 | *1BD9C328233CF457571A4BB5DB8D32892AB8EDBF |
| root | ::1       | *1BD9C328233CF457571A4BB5DB8D32892AB8EDBF |
+------+-----------+-------------------------------------------+
4 rows in set (0.00 sec)

mysql&gt;</pre>
<p>Time to copy the table back and reload the grant tables:</p>
<pre>[root@mysql mysql]# 'cp' test/user.MY* mysql/
[root@mysql mysql]# kill -SIGHUP $(pidof mysqld)</pre>
<p>And now you should be able to get back in:</p>
<pre>[root@mysql mysql]# mysql -p'newpass but this is insecure so dont use'
Warning: Using a password on the command line interface can be insecure.
Welcome to the MySQL monitor.  Commands end with ; or \g.
Your MySQL connection id is 35
Server version: 5.6.16 MySQL Community Server (GPL)

Copyright (c) 2000, 2014, Oracle and/or its affiliates. All rights reserved.

Oracle is a registered trademark of Oracle Corporation and/or its
affiliates. Other names may be trademarks of their respective
owners.

Type 'help;' or '\h' for help. Type '\c' to clear the current input statement.

mysql&gt; show grants;
+----------------------------------------------------------------------------------------------------------------------------------------+
| Grants for root@localhost                                                                                                              |
+----------------------------------------------------------------------------------------------------------------------------------------+
| GRANT ALL PRIVILEGES ON *.* TO 'root'@'localhost' IDENTIFIED BY PASSWORD '*0A131BF1166FB756A61317A40F272D6FFDD281E9' WITH GRANT OPTION |
| GRANT PROXY ON ''@'' TO 'root'@'localhost' WITH GRANT OPTION                                                                           |
+----------------------------------------------------------------------------------------------------------------------------------------+
2 rows in set (0.00 sec)</pre>
<p>There you go. We've regained root access to MySQL without restarting the service!</p>
<p>I hope you find this useful, and I'll leave opinions on MySQL's security as an exercise to the reader ...</p>
