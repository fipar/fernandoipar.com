---
layout: post
redirect_from:
  - 2016/11/04/mysql-can-be-evil/
  - 2016/11/04/mysql-can-be-evil.html

author: Fernando Ipar
status: publish
published: true
title: MySQL can be evil 
author: Fernando Ipar
author_login: fernando
author_email:
categories:
- planetmysql
- mysql
tags:
- mysql
---

Ok, maybe evil is too strong a word, but MySQL can certainly be deceiving. 

To some extent, and in some circles, MySQL still suffers from a bad reputation in terms of data integrity. In fact, I have personally tried to [fight that reputation](http://fernandoipar.com/mysql/2015/03/11/ongoing-mysql-myths.html) in the past. However, I'll be honest and say that, sometimes, it deserves it.

As an example, take the ASC|DESC keyword in index creation. [MySQL 8.0 support those](http://mysqlserverteam.com/mysql-8-0-labs-descending-indexes-in-mysql/), thanks for that! But what happens in older versions? Let me quote the [Fine Manual](http://dev.mysql.com/doc/refman/5.7/en/create-index.html):

       An index_col_name specification can end with ASC or DESC. These keywords are permitted for future extensions for
       specifying ascending or descending index value storage. Currently, they are parsed but ignored; index values are
       always stored in ascending order.

Pretty clear, and obvious to anyone who has been working with MySQL enough time. We just have this kind of knowledge almost built-in. So built-in, in fact, that it never occurred to me to check what happens when you create an index that way. So I put myself in the shoes of a DBA coming into MySQL from another engine today, and here's what our DBA would see:

       mysql> create index index_t_on_y on t(y desc);
       Query OK, 0 rows affected (0.02 sec)
       Records: 0  Duplicates: 0  Warnings: 0

And that's it. Not even a warning, and our DBA is left with the impression that, since the create index statement succeeded, producing no errors and no warnings, then a descending ordered index on y was created on table t. And we know how that impression will end.

Granted, this won't cause anyone to lose data, but still, things like this make me think that, sometimes, MySQL deserves its reputation.  