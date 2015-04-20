---
layout: post
redirect_from:
  - 2009/04/18/making-use-of-procedure-analyse/

status: publish
published: true
title: Making use of procedure analyse()
author:
  display_name: fernando
  login: fernando
  email: 
  url: http://fernandoipar.com
author_login: fernando
author_email: 
author_url: http://fernandoipar.com
wordpress_id: 155
wordpress_url: http://fernandoipar.com/?p=155
date: !binary |-
  MjAwOS0wNC0xOCAxNzo0Nzo0OCAtMDMwMA==
date_gmt: !binary |-
  MjAwOS0wNC0xOCAxOTo0Nzo0OCAtMDMwMA==
categories:
- MySQL
tags:
- MySQL
- optimization
- datatypes
comments:
- id: 42
  author: Roland Bouman
  author_email: Roland.Bouman@gmail.com
  author_url: http://rpbouman.blogspot.com/
  date: !binary |-
    MjAwOS0wNC0xOCAyMDowODowMCAtMDMwMA==
  date_gmt: !binary |-
    MjAwOS0wNC0xOCAyMjowODowMCAtMDMwMA==
  content: ! "Awesome! I never realized you could grab the output of a PROCEDURE ANALYSE()
    like that.\r\n\r\nThanks!"
- id: 43
  author: Arjen Lentz
  author_email: arjen@openquery.com
  author_url: http://openquery.com
  date: !binary |-
    MjAwOS0wNC0xOCAyMDoxNDo0MyAtMDMwMA==
  date_gmt: !binary |-
    MjAwOS0wNC0xOCAyMjoxNDo0MyAtMDMwMA==
  content: ! "Run ANALYSE(10,100) rather than without params. This sets the max# of
    ENUM items to 10 and a total of 100 bytes in the ENUM list.\r\nIn short, it gets
    rid of the nonsensical default of trying to stick everything in an enum.\r\n\r\nSee
    http://bugs.mysql.com/2049"
- id: 44
  author: Shlomi Noach
  author_email: shlomi@openark.org
  author_url: http://openark.org
  date: !binary |-
    MjAwOS0wNC0xOSAwMTozMDo1MCAtMDMwMA==
  date_gmt: !binary |-
    MjAwOS0wNC0xOSAwMzozMDo1MCAtMDMwMA==
  content: Well done!
- id: 45
  author: fernando
  author_email: mail@fernandoipar.com
  author_url: http://fernandoipar.com
  date: !binary |-
    MjAwOS0wNC0xOSAwMjoyNDoyNCAtMDMwMA==
  date_gmt: !binary |-
    MjAwOS0wNC0xOSAwNDoyNDoyNCAtMDMwMA==
  content: ! "Thanks for all the comments!\r\n\r\n@Arjen: I'll add these limits to
    my todo. I followed your link and couldn't help to laugh at the first reply you
    got. It's no wonder there are so many MySQL forks out there."
- id: 207
  author: Alex
  author_email: i2011@alexramos.net
  author_url: ''
  date: !binary |-
    MjAxMy0xMS0wMiAyMzoyODoxNiAtMDIwMA==
  date_gmt: !binary |-
    MjAxMy0xMS0wMyAwMjoyODoxNiAtMDIwMA==
  content: ! "Hi Fernando,\r\nThis seems like a great tool but I'm getting an error
    -\r\nERROR 1221 (HY000): Incorrect usage of PROCEDURE and non-SELECT\r\nI'm on
    the latest 5.6.x, maybe that's got something to do with it.\r\nAny ideas?\r\nThanks"
- id: 208
  author: fernando
  author_email: 
  author_url: http://fernandoipar.com
  date: !binary |-
    MjAxMy0xMS0wNCAwOToxMzoxOSAtMDIwMA==
  date_gmt: !binary |-
    MjAxMy0xMS0wNCAxMjoxMzoxOSAtMDIwMA==
  content: ! "Hello Alex, \r\n\r\nYou're right, this stopped working on 5.6 Not just
    the stored procedure, but apparently any use of procedure analyse() other than
    a select. \r\n\r\nSo far I have only found a changelog mention for 5.6.6 about
    this, but I tired 5.6.5 and this did not work either so the change must have happened
    before. \r\n\r\nI'll dig around a bit to see what I can find."
- id: 237
  author: root
  author_email: 23021278@qq.com
  author_url: ''
  date: !binary |-
    MjAxMy0xMi0wMSAwNDo1Nzo0MCAtMDIwMA==
  date_gmt: !binary |-
    MjAxMy0xMi0wMSAwNzo1Nzo0MCAtMDIwMA==
  content: ! "mysql&gt; insert into procedure_analyse_output select * from pet procedure
    analyse()\r\n    -&gt; ;\r\nERROR 1221 (HY000): Incorrect usage of PROCEDURE and
    non-SELECT"
- id: 238
  author: root
  author_email: 23021278@qq.com
  author_url: ''
  date: !binary |-
    MjAxMy0xMi0wMSAwNTowNjo0MSAtMDIwMA==
  date_gmt: !binary |-
    MjAxMy0xMi0wMSAwODowNjo0MSAtMDIwMA==
  content: ! "hello fernando,\r\nit can work this way:\r\nmysql&gt; select * from
    pet procedure analyse() into OUTFILE \"ana.txt\";\r\nmysql&gt; load data infile
    \"ana.txt\" into procedure_analyse_output;\r\nthan you la"
- id: 251
  author: 用PROCEDURE ANALYSE优化MYSQL表结构 | 青鬆下的ミ坚躯
  author_email: ''
  author_url: http://www.itxingzhe.com/168.html
  date: !binary |-
    MjAxNC0wMS0xMSAwMDoyNDo1OSAtMDIwMA==
  date_gmt: !binary |-
    MjAxNC0wMS0xMSAwMzoyNDo1OSAtMDIwMA==
  content: ! '[&#8230;] Making use of procedure analyse [&#8230;]'
---
<p>SELECT Field0[,Field1,Field2,...] FROM TABLE PROCEDURE ANALYSE() is a nice tool to find out more about your table's columns.</p>
<p>Still, it could be improved in a lot of ways, and the stored procedure below is a starting point. It makes use of <em>procedure analyse</em> (though with 'SELECT * FROM'), and modifies it's output to include the actual column datatype and the total number of rows of the table.</p>
<p>The actual datatype is a piece of information I've seen a lot of people request, and the number of rows is, I think, a critical piece of information to determine if the output of <em>procedure analyse</em> is credible or not. It's not the same thing to take suggestions from mysql on a table with 7 or 20 rows than from a table with 1000000 rows. Of course, remember than numbers alone mean nothing, you might just have 7 rows in a table that represent the entire universe of possible values for your problem domain. In any case, a smart human being with a good toolset is the best way to solve problems!</p>
<p>So here's the procedure, which can also be downloaded from this <a title="extended_procedure_analyse.sql" href="http://fernandoipar.com/extended_procedure_analyse.sql">link</a>:</p>
<pre>/*

extended procedure analyse
(C) 2009 Fernando Ipar
mail(at)fernandoipar.com

GPLv2

*/
drop procedure if exists extended_procedure_analyse;
delimiter //
create procedure extended_procedure_analyse(databaseName varchar(64), tableName varchar(64))
begin

	create temporary table procedure_analyse_output
	(
	Field_name varchar(64),
	Min_value int,
	Max_value int,
	Min_length int,
	Max_length int,
	Empties_or_zeros int,
	Nulls int,
	Avg_value_or_avg_length float,
	Std float,
	Optimal_fieldtype text,
	Actual_fieldtype text
	);	

	set @table = concat(databaseName,'.',tableName);
	set @dbName = databaseName;
	set @tbName = tableName;

	set @qry = concat('insert into procedure_analyse_output (Field_name,Min_value,Max_value,Min_length,Max_length,Empties_or_zeros,Nulls,Avg_Value_or_avg_length,Std,Optimal_fieldtype) select * from ', @table,' procedure analyse()');
	prepare myStmt from @qry;
	execute myStmt;

	update procedure_analyse_output set Field_name = replace(Field_name,  CONCAT(databaseName,'.',tableName,'.'),'');

	prepare myStmt from 'update procedure_analyse_output pao, information_schema.columns c set pao.Actual_Fieldtype = c.column_type where table_schema = ? and table_name = ? and column_name = pao.Field_name';
	execute myStmt using @dbName,@tbName;

	set @qry = concat('select count(*) as `Total_number_of_rows` from ',@table);
	prepare myStmt from @qry;

	execute myStmt;

	select * from procedure_analyse_output;

	drop temporary table procedure_analyse_output;

end;
//
delimiter ;</pre>
<p>Here are a couple of sample outputs:</p>
<pre>mysql&gt; call extended_procedure_analyse('test','City')\G
*************************** 1. row ***************************
Total_number_of_rows: 30000
1 row in set (0.11 sec)

*************************** 1. row ***************************
             Field_name: ID
              Min_value: 925001
              Max_value: 955000
             Min_length: 6
             Max_length: 6
       Empties_or_zeros: 0
                  Nulls: 0
Avg_value_or_avg_length: 940000
                    Std: 938839
      Optimal_fieldtype: MEDIUMINT(6) UNSIGNED NOT NULL
       Actual_fieldtype: int(11) unsigned
*************************** 2. row ***************************
             Field_name: CountryCode
              Min_value: 0
              Max_value: 29
             Min_length: 1
             Max_length: 2
       Empties_or_zeros: 1000
                  Nulls: 0
Avg_value_or_avg_length: 14.5
                    Std: 8.6554
      Optimal_fieldtype: ENUM('0','1','2','3','4','5','6','7','8','9','10','11','12','13','14','15','16','17','18','19','20','21','22','23','24','25','26','27','28','29') NOT NULL
       Actual_fieldtype: int(11) unsigned
*************************** 3. row ***************************
             Field_name: Name
              Min_value: 1
              Max_value: 9999
             Min_length: 1
             Max_length: 5
       Empties_or_zeros: 0
                  Nulls: 0
Avg_value_or_avg_length: 4.6605
                    Std: NULL
      Optimal_fieldtype: CHAR(5) NOT NULL
       Actual_fieldtype: varchar(40)
*************************** 4. row ***************************
             Field_name: District
              Min_value: 1
              Max_value: 9999
             Min_length: 1
             Max_length: 5
       Empties_or_zeros: 0
                  Nulls: 0
Avg_value_or_avg_length: 4.6603
                    Std: NULL
      Optimal_fieldtype: CHAR(5) NOT NULL
       Actual_fieldtype: varchar(40)
*************************** 5. row ***************************
             Field_name: Population
              Min_value: 0
              Max_value: 9999
             Min_length: 1
             Max_length: 5
       Empties_or_zeros: 0
                  Nulls: 0
Avg_value_or_avg_length: 4.6647
                    Std: NULL
      Optimal_fieldtype: CHAR(5) NOT NULL
       Actual_fieldtype: varchar(40)
5 rows in set (0.12 sec)

Query OK, 0 rows affected (0.12 sec)

mysql&gt; call extended_procedure_analyse('test','projects_innodb')\G
*************************** 1. row ***************************
Total_number_of_rows: 1007366
1 row in set (14.80 sec)

*************************** 1. row ***************************
             Field_name: id
              Min_value: 1
              Max_value: 1007366
             Min_length: 1
             Max_length: 7
       Empties_or_zeros: 0
                  Nulls: 0
Avg_value_or_avg_length: 503684
                    Std: 581599
      Optimal_fieldtype: MEDIUMINT(7) UNSIGNED NOT NULL
       Actual_fieldtype: int(10) unsigned
*************************** 2. row ***************************
             Field_name: name
              Min_value: 0
              Max_value: 9999
             Min_length: 1
             Max_length: 10
       Empties_or_zeros: 0
                  Nulls: 0
Avg_value_or_avg_length: 4.6958
                    Std: NULL
      Optimal_fieldtype: VARCHAR(10) NOT NULL
       Actual_fieldtype: char(10)
2 rows in set (14.80 sec)

Query OK, 0 rows affected (14.80 sec)

mysql&gt; call extended_procedure_analyse('test','projects_isam')\G
*************************** 1. row ***************************
Total_number_of_rows: 1000000
1 row in set (0.56 sec)

*************************** 1. row ***************************
             Field_name: id
              Min_value: 1
              Max_value: 1000000
             Min_length: 1
             Max_length: 7
       Empties_or_zeros: 0
                  Nulls: 0
Avg_value_or_avg_length: 500000
                    Std: 577358
      Optimal_fieldtype: MEDIUMINT(7) UNSIGNED NOT NULL
       Actual_fieldtype: int(10) unsigned
*************************** 2. row ***************************
             Field_name: name
              Min_value: 0
              Max_value: 9999
             Min_length: 1
             Max_length: 5
       Empties_or_zeros: 0
                  Nulls: 0
Avg_value_or_avg_length: 4.6605
                    Std: NULL
      Optimal_fieldtype: CHAR(5) NOT NULL
       Actual_fieldtype: char(10)
2 rows in set (0.56 sec)

Query OK, 0 rows affected (0.56 sec)</pre>
<p>Notice the difference in response time between the Innodb and MyISAM (yes, I wrongly used the 'isam' name ...) tables, that's because Innodb has to calculate the number of rows for a count(*) query, while MyISAM stores a row count in the table.</p>
<p>I plan to extend this procedure to include index information, with useful data such as overindexed columns (columns that are included as a leftmost prefix in more than one index) and unindexed columns that are queried. The first question is easily answered from information_schema.statistics, using the seq_in_index column, I'm working on the second one.</p>
<p>Still, if you have the time and interest, play with this early version and let me know what's wrong and/or could be improved with it.</p>
