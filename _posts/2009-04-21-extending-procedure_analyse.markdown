---
layout: post
redirect_from:
  - 2009/04/21/extending-procedure_analyse/

author: Fernando Ipar
status: publish
published: true
title: Extending procedure_analyse
author_login: fernando
author_email: 
wordpress_id: 163
date: !binary |-
  MjAwOS0wNC0yMSAxNDowNjoyNiAtMDMwMA==
date_gmt: !binary |-
  MjAwOS0wNC0yMSAxNjowNjoyNiAtMDMwMA==
categories:
- Programming
- MySQL
tags: []
comments:
- id: 46
  author: Shlomi Noach
  author_email: shlomi@openark.org
  author_url: http://openark.org
  date: !binary |-
    MjAwOS0wNC0yMiAwMToyMjo1MSAtMDMwMA==
  date_gmt: !binary |-
    MjAwOS0wNC0yMiAwMzoyMjo1MSAtMDMwMA==
  content: ! "Good work.\r\n\r\nWith regard to overused indexes, you should note that
    an index on (a,b) and on (a,c) have nothing useless between them. But your code
    will report \"a\" as being over indexed."
- id: 47
  author: Fernando Ipar
  author_email: 
  date: !binary |-
    MjAwOS0wNC0yMiAwMTo0OToxMyAtMDMwMA==
  date_gmt: !binary |-
    MjAwOS0wNC0yMiAwMzo0OToxMyAtMDMwMA==
  content: ! "Thanks a lot for the observation, that didn't occur to me :)\r\n\r\nI'll
    have to look just for columns that have count(*) &gt; 1 and that are also indexed
    alone, this would be the useless index (of course, it might be that one index
    takes duplicates, the other not, I don't know if that still makes all cases useless,
    you've really opened a can of worms for me :)\r\n\r\nFortunately we've got the
    thing at bitbucket now so I'll report this as a bug and begin working on it ASAP!"
- id: 48
  author: Shlomi Noach
  author_email: shlomi@openark.org
  author_url: http://openark.org
  date: !binary |-
    MjAwOS0wNC0yMiAwMTo1ODo1NCAtMDMwMA==
  date_gmt: !binary |-
    MjAwOS0wNC0yMiAwMzo1ODo1NCAtMDMwMA==
  content: ! "Hi,\r\nStill not complete. If the the index on (a) is UNIQUE, then an
    index (a,b) does make it redundant.\r\nSince there's already a utility which checks
    for duplicate indexes, and it's quite complicated, I suggest you take a look how
    it does it. It is part of maatkit, and is called mk-duplicate-key-checker. See
    <a href=\"http://code.google.com/p/maatkit/wiki/DeterminingDuplicateKeys\" rel=\"nofollow\">http://code.google.com/p/maatkit/wiki/DeterminingDuplicateKeys</a>"
- id: 49
  author: Fernando Ipar
  author_email: 
  date: !binary |-
    MjAwOS0wNC0yMiAwMjozMzoyMCAtMDMwMA==
  date_gmt: !binary |-
    MjAwOS0wNC0yMiAwNDozMzoyMCAtMDMwMA==
  content: Thanks for the additional suggestion, as I said, I expected that kind of
    problems. I'll look into that particular maatkit script for ideas, I don't intend
    to reinvent the wheel but rather to present DBAs with some useful screenshot of
    information, but it has to be useful and properly obtained :)
---
<p>My previous post explored a stored procedure that extended procedure_analyse with the intent of helping DBAs optimize table structure.</p>
<p>Here's an improved version. I've followed <a title="Open Query" href="http://openquery.com/">Arjen Lentz</a>'s suggestion and added support for the max_elements and max_memory parameters.</p>
<p>I also added a new Indexed column to the output, which is an ENUM('No','Yes','Overindexed'). Yes and No are self-explanatory, while Overindexed means the column is present as the left-most part of more than one index. This is useless, it just presents a performance penalty for MySQL (it needs to update more indexes) and if, for instance, you have columns A and B, and you have KEY(A) and KEY (A,B), mysql can use the second index to search for A alone too.</p>
<p>Here's the updated version:</p>
<pre>
/*

extended procedure analyse
(C) 2009 Fernando Ipar
mail(at)fernandoipar.com

GPLv2

*/
drop procedure if exists extended_procedure_analyse;
delimiter //
create procedure extended_procedure_analyse(databaseName varchar(64), tableName varchar(64), max_elements int, max_memory int)
begin

	drop temporary table if exists procedure_analyse_output;
	drop temporary table if exists tmp_pao;

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
	Actual_fieldtype text,
	Indexed enum ('No','Yes','Overindexed') default 'No'
	);	
	
	set @table = concat(databaseName,'.',tableName);
	set @dbName = databaseName;
	set @tbName = tableName;
	set @maxEle = max_elements;
	set @maxMem = max_memory;

	set @qry = concat('insert into procedure_analyse_output (Field_name,Min_value,Max_value,Min_length,Max_length,Empties_or_zeros,Nulls,Avg_Value_or_avg_length,Std,Optimal_fieldtype) select * from ', @table,' procedure analyse(',@maxEle,',',@maxMem,')');
	prepare myStmt from @qry;
	execute myStmt;
	
	update procedure_analyse_output set Field_name = replace(Field_name,  CONCAT(databaseName,'.',tableName,'.'),'');
	
	prepare myStmt from 'update procedure_analyse_output pao, information_schema.columns c set pao.Actual_Fieldtype = c.column_type where table_schema = ? and table_name = ? and column_name = pao.Field_name';
	execute myStmt using @dbName,@tbName;
	
	set @qry = concat('select count(*) as `Total_number_of_rows` from ',@table);
	prepare myStmt from @qry;
	
	execute myStmt;
	
	set @qry = concat('update procedure_analyse_output pao set Indexed = "Yes\" where exists (select 1 from information_schema.statistics where table_schema = ? and table_name = ? and column_name = pao.Field_name)');
	prepare myStmt from @qry;
	execute myStmt using @dbName,@tbName;
	

	create temporary table tmp_pao as select * from procedure_analyse_output;
	prepare myStmt from 'update tmp_pao set Indexed = "Overindexed" where exists (select Field_name,count(*) from procedure_analyse_output pao inner join information_schema.statistics s on pao.Field_name = s.column_name where table_schema = ? and table_name = ? and seq_in_index = 1 and pao.Field_name = tmp_pao.Field_name group by Field_name having count(*) > 1)';
	execute myStmt  using @dbName,@tbName;
		
	select * from tmp_pao;
	
	drop temporary table procedure_analyse_output;
	drop temporary table tmp_pao; 
	
end;
//
delimiter ;

</pre>
<p>And here's a sample output: </p>
<pre>
mysql> call extended_procedure_analyse('test','Account',4,100)\G
*************************** 1. row ***************************
Total_number_of_rows: 1
1 row in set (0.04 sec)

*************************** 1. row ***************************
             Field_name: InternalAID
              Min_value: 2147483647
              Max_value: 2147483647
             Min_length: 18
             Max_length: 18
       Empties_or_zeros: 0
                  Nulls: 0
Avg_value_or_avg_length: 1.65632e+17
                    Std: 0
      Optimal_fieldtype: BIGINT(18) UNSIGNED NOT NULL
       Actual_fieldtype: bigint(20)
                Indexed: Yes
*************************** 2. row ***************************
             Field_name: accountID
              Min_value: 12
              Max_value: 12
             Min_length: 2
             Max_length: 2
       Empties_or_zeros: 0
                  Nulls: 0
Avg_value_or_avg_length: 2
                    Std: NULL
      Optimal_fieldtype: ENUM('12') NOT NULL
       Actual_fieldtype: varchar(255)
                Indexed: Overindexed
*************************** 3. row ***************************
             Field_name: acctBalance
              Min_value: 2147483647
              Max_value: 2147483647
             Min_length: 11
             Max_length: 11
       Empties_or_zeros: 0
                  Nulls: 0
Avg_value_or_avg_length: 1.00051e+20
                    Std: 0
      Optimal_fieldtype: BIGINT(11) UNSIGNED NOT NULL
       Actual_fieldtype: double
                Indexed: Yes
*************************** 4. row ***************************
             Field_name: ownerID
              Min_value: 2147483647
              Max_value: 2147483647
             Min_length: 19
             Max_length: 19
       Empties_or_zeros: 0
                  Nulls: 0
Avg_value_or_avg_length: 6.57541e+18
                    Std: 0
      Optimal_fieldtype: BIGINT(19) UNSIGNED NOT NULL
       Actual_fieldtype: bigint(20)
                Indexed: No
4 rows in set (0.24 sec)

Query OK, 0 rows affected, 8 warnings (0.24 sec)

</pre>
<p>There's a <a title="mydbsuggest at bitbucket" href="http://www.bitbucket.org/nandix/mydbsuggest/">hg repository</a> to handle the project. We're working with <a title="fedesilva@bitbucket.org" href="http://www.bitbucket.org/fedesilva">@fedesilva</a> to create a standalone java app that will present this and more info in a friendly manner to sysadmins, and to handle the creation and destruction of the stored procedure automatically.</p>
