---
layout: post
redirect_from:
  - 2009/03/09/using-the-enum-data-type-to-increase-performance/

author: Fernando Ipar
status: publish
published: true
title: Using the ENUM data type to increase performance
author_login: fernando
author_email: 
excerpt: ! "While going through the DATA TYPES section of the Certification Study
  Guide, I was refreshed of the ENUM datatype, which I rarely use. \r\n\r\nI usually
  create individual tables for enumerations, so that new values can be added with
  just an insert, or deprecated values can be marked as such. \r\n\r\nHowever, today
  I got to think about the performance issues involved in all that joining, and how
  could an ENUM column improve a select. "
wordpress_id: 119
date: !binary |-
  MjAwOS0wMy0wOSAxMTo0ODo0NCAtMDMwMA==
date_gmt: !binary |-
  MjAwOS0wMy0wOSAxMzo0ODo0NCAtMDMwMA==
categories:
- MySQL
tags:
- MySQL
- performance
comments:
- id: 110
  author: chris
  author_email: shadow_shdw2000@yahoo.com
  author_url: ''
  date: !binary |-
    MjAwOS0xMS0xMyAwOToxOTo1NSAtMDIwMA==
  date_gmt: !binary |-
    MjAwOS0xMS0xMyAxMjoxOTo1NSAtMDIwMA==
  content: i hope that u continue on posting more samples of programs... thnx!
- id: 111
  author: Fernando
  author_email: 
  date: !binary |-
    MjAwOS0xMS0xMyAxMDo1Mzo0MSAtMDIwMA==
  date_gmt: !binary |-
    MjAwOS0xMS0xMyAxMzo1Mzo0MSAtMDIwMA==
  content: ! "Thanks for the message chris. \r\nI've been neglecting the blog due
    to work lately, but I'm sure coming back to it. \r\nI'm standing by for OpenSQL
    Camp in Portland with my Percona team mates right now, that should give me some
    nice blogging material for my return :)"
- id: 143
  author: Robert
  author_email: robert@xarg.org
  author_url: http://www.xarg.org/
  date: !binary |-
    MjAxMS0wMi0yNiAwNjo1MzozMSAtMDIwMA==
  date_gmt: !binary |-
    MjAxMS0wMi0yNiAwOTo1MzozMSAtMDIwMA==
  content: ! "Fernando,\r\n\r\nI wonder if you would get the same performance results
    if you exhaust the enum data type with up to 2^16-1 elements. I mean, what happens,
    if you have many different types, would it be better to pre-define all possible
    combinations in the scheme or would it be better to use a normalized scheme? Just
    from a performance point of view of course, elegance is somewhat different :D\r\n\r\nBut
    I think the difference would be negigible because you only shift the lookup problem.\r\n\r\nRobert"
- id: 1576
  author: Tony
  author_email: mahzetest2@gmail.com
  author_url: http://codenevis.ir
  date: !binary |-
    MjAxNC0wOS0yMyAwNDo0MDo1NiAtMDMwMA==
  date_gmt: !binary |-
    MjAxNC0wOS0yMyAwNzo0MDo1NiAtMDMwMA==
  content: I always experienced lots of problems using MySQL ENUMs , and never going
    to use it again.
---
<p>While going through the DATA TYPES section of the Certification Study Guide, I was refreshed of the ENUM datatype, which I rarely use. </p>
<p>I usually create individual tables for enumerations, so that new values can be added with just an insert, or deprecated values can be marked as such. </p>
<p>However, today I got to think about the performance issues involved in all that joining, and how could an ENUM column improve a select. </p>
<p>Here's what I came up with: </p>
<pre>
mysql> create table project_types (
    -> id int unsigned not null auto_increment,
    -> name char(30) not null,
    -> primary key (id),
    -> index(name)
    -> ) Engine InnoDB;
Query OK, 0 rows affected (0.00 sec)
</pre>
<p>create the projects table</p>
<pre>
mysql> create table projects (
    -> id int unsigned not null auto_increment,
    -> name char(30) not null,
    -> project_type int unsigned not null,
    -> primary key (id),
    -> index(name),
    -> constraint `proyects_project_type` foreign key (`project_type`) references project_types(`id`)
    -> ) Engine InnoDB;
Query OK, 0 rows affected (0.01 sec)
</pre>
<p>insert some project types</p>
<pre>
mysql> select * from project_types order by id;
+----+-------------+
| id | name        |
+----+-------------+
|  1 | Development | 
|  2 | Consultancy | 
|  3 | Research    | 
|  4 | Support     | 
+----+-------------+
4 rows in set (0.00 sec)
</pre>
<p>insert several projects. </p>
<pre>
i=0; while [ $i -lt 100000000 ]; do echo "'$RANDOM',$((($RANDOM % 4)+1))">> projects.csv; i=$((i+1)); done
</pre>
<p>I'm not patient, so I cancelled the process somewhere in the middle, then did a few cat's of<br />
the file to get more records, and then loaded the resulting file into mysql with a simple: </p>
<pre>
mysql> load data infile '/tmp/fullprojects.csv' into table projects fields terminated by ',' (name,project_type);
</pre>
<p>Stopped it sometime after a few hours :)<br />
I ended up with 1.774.292 rows, which I hope will be enough for testing. </p>
<pre>

mysql> set profiling = 1;
Query OK, 0 rows affected (0.00 sec)


mysql> select projects.name as Project, project_types.name as Type from projects,project_types where project_types.id = projects.project_type limit 10000 into outfile '/tmp/projects.txt';
Query OK, 10000 rows affected (2.26 sec)

mysql> show profile for query 1;
+--------------------------------+----------+
| Status                         | Duration |
+--------------------------------+----------+
| (initialization)               | 0.000005 | 
| checking query cache for query | 0.000135 | 
| Opening tables                 | 0.000214 | 
| System lock                    | 0.000016 | 
| Table lock                     | 0.000024 | 
| init                           | 0.000255 | 
| optimizing                     | 0.000027 | 
| statistics                     | 0.000057 | 
| preparing                      | 0.000028 | 
| executing                      | 0.000009 | 
| Sending data                   | 2.263521 | 
| end                            | 0.000027 | 
| query end                      | 0.000014 | 
| freeing items                  | 0.000025 | 
| closing tables                 | 0.000021 | 
| logging slow query             | 0.000126 | 
+--------------------------------+----------+
16 rows in set (0.00 sec)

mysql> select projects.name as Project, project_types.name as Type from projects,project_types where project_types.id = projects.project_type limit 100000 into outfile '/tmp/projects.txt';
Query OK, 100000 rows affected (10.23 sec)

mysql> show profile for query 2;
+--------------------------------+-----------+
| Status                         | Duration  |
+--------------------------------+-----------+
| (initialization)               | 0.000012  | 
| checking query cache for query | 0.000128  | 
| Opening tables                 | 0.000031  | 
| System lock                    | 0.00001   | 
| Table lock                     | 0.000021  | 
| init                           | 0.000241  | 
| optimizing                     | 0.000026  | 
| statistics                     | 0.000056  | 
| preparing                      | 0.000029  | 
| executing                      | 0.000008  | 
| Sending data                   | 10.224004 | 
| end                            | 0.000025  | 
| query end                      | 0.00001   | 
| freeing items                  | 0.000023  | 
| closing tables                 | 0.00002   | 
| logging slow query             | 0.000087  | 
+--------------------------------+-----------+
16 rows in set (0.00 sec)


mysql> select projects.name as Project, project_types.name as Type from projects,project_types where project_types.id = projects.project_type limit 1000000 into outfile '/tmp/projects.txt';
Query OK, 1000000 rows affected (6 min 7.36 sec)

mysql> show profile for query 3;
+--------------------------------+-----------+
| Status                         | Duration  |
+--------------------------------+-----------+
| (initialization)               | 0.000011  | 
| checking query cache for query | 0.000138  | 
| Opening tables                 | 0.000032  | 
| System lock                    | 0.000132  | 
| Table lock                     | 0.000024  | 
| init                           | 0.000375  | 
| optimizing                     | 0.000027  | 
| statistics                     | 0.000079  | 
| preparing                      | 0.000171  | 
| executing                      | 0.000009  | 
| Sending data                   | 379.69978 | 
| end                            | 0.000013  | 
| query end                      | 0.000006  | 
| freeing items                  | 0.000013  | 
| closing tables                 | 0.000009  | 
| logging slow query             | 0.0000800 | 
+--------------------------------+-----------+
16 rows in set (0.01 sec)



</pre>
<p>Now, a version of the projects table using the ENUM data type: </p>
<pre>

mysql> select p.name,pt.name from projects p, project_types pt where pt.id = p.project_type into outfile '/tmp/projects.csv' fields terminated by ',';
Query OK, 1774292 rows affected (12 min 3.68 sec)

mysql> create table projects_unnormalized (name char(30) not null, project_type enum ('Consultancy','Development','Research','Support'));
Query OK, 0 rows affected (0.01 sec)

mysql> load data infile '/tmp/projects.csv' into table projects_unnormalized fields terminated by ',';
Query OK, 1774292 rows affected (2.56 sec)
Records: 1774292  Deleted: 0  Skipped: 0  Warnings: 0

mysql> alter table projects_unnormalized add id int unsigned not null auto_increment primary key;
Query OK, 1774292 rows affected (9.53 sec)
Records: 1774292  Duplicates: 0  Warnings: 0

mysql> alter table projects_unnormalized add index(name), add index(project_type);
Query OK, 1774292 rows affected (30.96 sec)
Records: 1774292  Duplicates: 0  Warnings: 0


mysql> select projects_unnormalized.name as Project, projects_unnormalized.project_type as Type from projects_unnormalized limit 10000 into outfile '/tmp/projects.txt';
Query OK, 10000 rows affected (0.01 sec)

mysql> show profile for query 75;
+--------------------------------+----------+
| Status                         | Duration |
+--------------------------------+----------+
| (initialization)               | 0.000013 | 
| checking query cache for query | 0.000122 | 
| Opening tables                 | 0.000023 | 
| System lock                    | 0.000012 | 
| Table lock                     | 0.000021 | 
| init                           | 0.00023  | 
| optimizing                     | 0.000012 | 
| statistics                     | 0.000026 | 
| preparing                      | 0.000021 | 
| executing                      | 0.000009 | 
| Sending data                   | 0.016887 | 
| end                            | 0.000024 | 
| query end                      | 0.00001  | 
| freeing items                  | 0.000018 | 
| closing tables                 | 0.000017 | 
| logging slow query             | 0.000081 | 
+--------------------------------+----------+
16 rows in set (0.00 sec)

mysql> select projects_unnormalized.name as Project, projects_unnormalized.project_type as Type from projects_unnormalized limit 100000 into outfile '/tmp/projects.txt';
Query OK, 100000 rows affected (0.17 sec)

mysql> show profile for query 77;
+--------------------------------+-----------+
| Status                         | Duration  |
+--------------------------------+-----------+
| (initialization)               | 0.000015  | 
| checking query cache for query | 0.00013   | 
| Opening tables                 | 0.000027  | 
| System lock                    | 0.000011  | 
| Table lock                     | 0.00002   | 
| init                           | 0.000236  | 
| optimizing                     | 0.000012  | 
| statistics                     | 0.000026  | 
| preparing                      | 0.000021  | 
| executing                      | 0.000008  | 
| Sending data                   | 0.177526  | 
| end                            | 0.000022  | 
| query end                      | 0.00001   | 
| freeing items                  | 0.000022  | 
| closing tables                 | 0.000021  | 
| logging slow query             | 0.0000900 | 
+--------------------------------+-----------+
16 rows in set (0.00 sec)


mysql> select projects_unnormalized.name as Project, projects_unnormalized.project_type as Type from projects_unnormalized limit 1000000 into outfile '/tmp/projects.txt';
Query OK, 1000000 rows affected (0.85 sec)

mysql> show profile for query 79;
+--------------------------------+-----------+
| Status                         | Duration  |
+--------------------------------+-----------+
| (initialization)               | 0.000005  | 
| checking query cache for query | 0.000128  | 
| Opening tables                 | 0.000031  | 
| System lock                    | 0.000011  | 
| Table lock                     | 0.00002   | 
| init                           | 0.000244  | 
| optimizing                     | 0.000012  | 
| statistics                     | 0.0000950 | 
| preparing                      | 0.000024  | 
| executing                      | 0.000007  | 
| Sending data                   | 2.605529  | 
| end                            | 0.000023  | 
| query end                      | 0.000008  | 
| freeing items                  | 0.00002   | 
| closing tables                 | 0.00002   | 
| logging slow query             | 0.000098  | 
+--------------------------------+-----------+
16 rows in set (0.00 sec)

</pre>
<p>And now, EXPLAIN for both schemas: </p>
<pre>

mysql> explain select projects.name as Project, project_types.name as Type from projects,project_types where project_types.id = projects.project_type;
+----+-------------+---------------+-------+-----------------------+-----------------------+---------+-----------------------+--------+-------------+
| id | select_type | table         | type  | possible_keys         | key                   | key_len | ref                   | rows   | Extra       |
+----+-------------+---------------+-------+-----------------------+-----------------------+---------+-----------------------+--------+-------------+
|  1 | SIMPLE      | project_types | index | PRIMARY               | name                  | 30      | NULL                  |      4 | Using index | 
|  1 | SIMPLE      | projects      | ref   | proyects_project_type | proyects_project_type | 4       | test.project_types.id | 512503 |             | 
+----+-------------+---------------+-------+-----------------------+-----------------------+---------+-----------------------+--------+-------------+
2 rows in set (0.00 sec)

mysql> explain select projects_unnormalized.name as Project, projects_unnormalized.project_type as Type from projects_unnormalized;
+----+-------------+-----------------------+------+---------------+------+---------+------+---------+-------+
| id | select_type | table                 | type | possible_keys | key  | key_len | ref  | rows    | Extra |
+----+-------------+-----------------------+------+---------------+------+---------+------+---------+-------+
|  1 | SIMPLE      | projects_unnormalized | ALL  | NULL          | NULL | NULL    | NULL | 1774292 |       | 
+----+-------------+-----------------------+------+---------------+------+---------+------+---------+-------+
1 row in set (0.00 sec)

</pre>
<p>Well, it's quite obvious that while using ENUM might seem a little less elegant from a design point of view (what happens if I use Types in more than just one table? What if I want to add a type? I have to alter the table!), the performance benefits, if I'm going to be handling large quantities of data (and my tests have been with small amounts, but I don't have a server, just a very humble notebook) might be worthwhile. </p>
<p>Someetimes, it's OK to bend the rules a little bit if you need a performance boost. Just like GOTO loops are sometimes OK (mysql has some on it's source code, btw). It's all a matter of need, expertise, and experience. </p>
<p>The TAO of programming puts it better: </p>
<blockquote><p>
There once was a master programmer who wrote unstructured programs. A novice programmer, seeking to imitate him, also began to write unstructured programs. When the novice asked the master to evaluate his progress, the master criticized him for writing unstructured programs, saying, ``What is appropriate for the master is not appropriate for the novice. You must understand the Tao before transcending structure.''
</p></blockquote>
<p>It should be clear from the output of EXPLAIN that going through two tables, doing a join with of ref type with that many tuples is probably not good (At least that's what it says in the manual: "If the key that is used matches only a few rows, this is a good join type"), while the table structure using the ENUM datatype can return all rows at the cost of a single table scan. </p>
<p>The main reason is that, according to my understanding, the ENUM datatype implements the mapping between the byte that's stored in the table to reference the element and the mnemonic name you give to it using the internal structure known as typelib. </p>
<p>You can refer to lines 7771 - 7782 of libmysqld/field.cc in the MySQL CE 5.0 source code for the definition of the val_str in the Field_enum data type to find this, here's the snippet: </p>
<pre>
String *Field_enum::val_str(String *val_buffer __attribute__((unused)),
                            String *val_ptr)
{
  uint tmp=(uint) Field_enum::val_int();
  if (!tmp || tmp > typelib->count)
    val_ptr->set("", 0, field_charset);
  else
    val_ptr->set((const char*) typelib->type_names[tmp-1],
                 typelib->type_lengths[tmp-1],
                 field_charset);
  return val_ptr;
}

</pre>
<p>Well, this is a very simple example, but it's all I needed today :)</p>
