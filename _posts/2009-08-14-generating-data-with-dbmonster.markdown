---
layout: post
redirect_from:
  - 2009/08/14/generating-data-with-dbmonster/

author: Fernando Ipar
status: publish
published: true
title: Generating data with dbmonster
author_login: fernando
author_email: 
wordpress_id: 182
date: !binary |-
  MjAwOS0wOC0xNCAxODowMjo1NiAtMDMwMA==
date_gmt: !binary |-
  MjAwOS0wOC0xNCAyMDowMjo1NiAtMDMwMA==
categories:
- MySQL
tags:
- MySQL
- Testing
- Java
comments:
- id: 73
  author: pabloj
  author_email: pmagnoli@gmail.com
  author_url: ''
  date: !binary |-
    MjAwOS0wOC0xNiAxMjozMToyOSAtMDMwMA==
  date_gmt: !binary |-
    MjAwOS0wOC0xNiAxNDozMToyOSAtMDMwMA==
  content: ! "Sorry but:\r\n\r\n1. it can handle referential integrity\r\n2. you don't
    have to create the XML file, just use schema grabber\r\n\r\nSee http://pabloj.blogspot.com/2005/10/feeding-your-database.html"
- id: 74
  author: fernando
  author_email: mail@fernandoipar.com
  date: !binary |-
    MjAwOS0wOC0xNiAxMzozMDoyNyAtMDMwMA==
  date_gmt: !binary |-
    MjAwOS0wOC0xNiAxNTozMDoyNyAtMDMwMA==
  content: ! "Hey, no need to be sorry, that's good news :)\r\nMy mistake was going
    straight to the configuration section of the manual. Since foreign keys are handles
    automatically according to this: http://dbmonster.kernelpanic.pl/manual/intro.html,
    there's no configuration key/option for them. \r\nI'll run new tests taking this
    into consideration and either update this post or write a new one. \r\n\r\nThanks
    for the info and the link!"
- id: 78
  author: fernando
  author_email: mail@fernandoipar.com
  date: !binary |-
    MjAwOS0wOC0xOCAxMTowMzowMiAtMDMwMA==
  date_gmt: !binary |-
    MjAwOS0wOC0xOCAxNDowMzowMiAtMDMwMA==
  content: ! "Indeed, I ran another test and it works exactly as described: \r\n\r\nmysql
    -e 'create database rtest' \r\n[... create some inodb tables on rtest, with foreign
    key constraints ... ]\r\n\r\njava pl.kernelpanic.dbmonster.Launcher -c rtest.properties
    --grab &gt; rtest.xml\r\n\r\nAfter properly editing the begining of the rtest.xml
    file (since it will included some non-xml output from dbmonster): \r\n\r\njava
    pl.kernelpanic.dbmonster.Launcher -c rtest.properties -s rtest.xml\r\n\r\nThis
    successfully generates rows for all tables, satisfying referential integrity :)"
- id: 108
  author: Keyword stuffing, AdSense, Search engine optimization, and more. | Internet
    Marketing Dispatch
  author_email: ''
  author_url: http://www.internetmarketingdispatch.com/2009/08/14/keyword-stuffing-adsense-search-engine-optimization-and-more/
  date: !binary |-
    MjAwOS0xMC0yNiAwNzozNjowMSAtMDIwMA==
  date_gmt: !binary |-
    MjAwOS0xMC0yNiAxMDozNjowMSAtMDIwMA==
  content: ! '[...] Generating data with dbmonster on Planet MySQL [...]'
---
<p>
In my <a href="http://fernandoipar.com/2009/08/12/indexing-text-columns-in-mysql/">last post</a> I included some sample data which was useful for playing around with queries </p>
<p>
That sample data was generated with dbmonster, a nice tool I discovered recently which comes in handy when you need to populate a database table to test your queries, the engine, schema, etc.</p>
<p>
In a nutshell, here are the strengths I like about it:</p>
<ul>
<li>It's written in Java, so it'll feed any database with a JDBC driver, and AFAIK, that's just about any RDBMS you might need to use</li>
<li>It's not too slow (it's actually pretty fast if you don't go crazy on long text based datatypes)</li>
<li>It includes several data generators, and being written in Java, rolling out my own should be easy. </li>
</ul>
<p>As for the weaknesses:</p>
<ul>
<li>It can be unbearably slow with long text columns</li>
<li>The generated text data might not represent a very real installation (again, look at my previous post for a quick sample). Still, the fact that you can easily write and plug your own generator(s) is enough to give kudos to the developers for a good architecture. Even if a feature isn't available now, it's painless to add in the future</li>
<li><del>It has no way to manage relational integrity, hence, it doesn't really feed RDBMS, just JDBC enabled databases</del> (yes it does, please see the comments)</li>
</ul>
<p>
<del>My last point means that, if you have table A and table B, both related through a foreign key constraint, dbmonster doesn't provide a way for you to generate keys for table A and then reuse these keys<br />
while inserting the foreign keys in table B. Not that I've found so far, at least.</del></p>
<p>
Related with this limitation, tables are generated one at the time. Still, if you need a tool to populate tables, even very large ones, with random values, heck, up until a few days ago, I was doing this from<br />
bash or python, so it's definitely an improvement. <del>I've discussed this with colleagues and some proprietary tools are either available or in the works that would handle referential integrity while<br />
generating data for database testing. If anyone knows an Open Source one that does, it'd be great. Otherwise, that would sure make one hell of a good project :)</del></p>
<p>
Ok, so enough rant, here's some data: You get it from <a href="http://dbmonster.kernelpanic.pl/">here</a>.</p>
<p>
Start off by creating a dbmonster.properties file. Here's a sample:</p>
<pre>
dbmonster.jdbc.driver=com.mysql.jdbc.Driver
dbmonster.jdbc.url=jdbc:mysql://localhost/sample_db
dbmonster.jdbc.username=mysql_user
dbmonster.jdbc.password=mysql_password

# for Oracle and other schema enabled databases
#dbmonster.jdbc.schema=schema_name

# maximal number of (re)tries
dbmonster.max-tries=50

# default rows number for SchemaGrabber
dbmonster.rows=1000

# progres monitor class
dbmonster.progress.monitor=pl.kernelpanic.dbmonster.ProgressMonitorAdapter
</pre>
<p>
Dbmonster is now ready to reach your database.<br />
Before it's ready to feed it, though, you need to create an xml file (ah, yes, the plague of the Java world, and I've been a Java guy for most of my working years...). Call it what you want,<br />
being a consistent kind type of man, I've been using <schema-name>.xml. Here's my sample_db.xml file:</p>
<pre>
&lt;?xml version="1.0"?&gt;
&lt;!DOCTYPE dbmonster-schema PUBLIC "-//kernelpanic.pl//DBMonster Database Schema DTD 1.1//EN" "http://dbmonster.kernelpanic.pl/dtd/dbmonster-schema-1.1.dtd"&gt;
&lt;dbmonster-schema&gt;
  &lt;name&gt;Sample DB&lt;/name&gt;
  &lt;table name="jobs" rows="150000"&gt;
    &lt;key databaseDefault="true"&gt;
      &lt;generator type="pl.kernelpanic.dbmonster.generator.MaxKeyGenerator"&gt;
        &lt;property name="columnName" value="id"/&gt;
      &lt;/generator&gt;
    &lt;/key&gt;
    &lt;column name="title" databaseDefault="false"&gt;
      &lt;generator type="pl.kernelpanic.dbmonster.generator.StringGenerator"&gt;
        &lt;property name="allowSpaces" value="true"/&gt;
        &lt;property name="excludeChars" value=""/&gt;
        &lt;property name="maxLength" value="250"/&gt;
        &lt;property name="minLength" value="15"/&gt;
        &lt;property name="nulls" value="0"/&gt;
      &lt;/generator&gt;
    &lt;/column&gt;
    &lt;column name="link" databaseDefault="false"&gt;
      &lt;generator type="pl.kernelpanic.dbmonster.generator.StringGenerator"&gt;
        &lt;property name="allowSpaces" value="true"/&gt;
        &lt;property name="excludeChars" value=""/&gt;
        &lt;property name="maxLength" value="65500"/&gt;
        &lt;property name="minLength" value="150"/&gt;
        &lt;property name="nulls" value="0"/&gt;
      &lt;/generator&gt;
    &lt;/column&gt;
    &lt;column name="description" databaseDefault="false"&gt;
      &lt;generator type="pl.kernelpanic.dbmonster.generator.StringGenerator"&gt;
        &lt;property name="allowSpaces" value="true"/&gt;
        &lt;property name="excludeChars" value=""/&gt;
        &lt;property name="maxLength" value="65500"/&gt;
        &lt;property name="minLength" value="150"/&gt;
        &lt;property name="nulls" value="0"/&gt;
      &lt;/generator&gt;
    &lt;/column&gt;
    &lt;column name="city" databaseDefault="false"&gt;
      &lt;generator type="pl.kernelpanic.dbmonster.generator.StringGenerator"&gt;
        &lt;property name="allowSpaces" value="true"/&gt;
        &lt;property name="excludeChars" value=""/&gt;
        &lt;property name="maxLength" value="250"/&gt;
        &lt;property name="minLength" value="15"/&gt;
        &lt;property name="nulls" value="0"/&gt;
      &lt;/generator&gt;
    &lt;/column&gt;
    &lt;column name="postdate" databaseDefault="false"&gt;
      &lt;generator type="pl.kernelpanic.dbmonster.generator.DateTimeGenerator"&gt;
        &lt;property name="nulls" value="0"/&gt;
      &lt;/generator&gt;
    &lt;/column&gt;
    &lt;column name="company_id" databaseDefault="false"&gt;
      &lt;generator type="pl.kernelpanic.dbmonster.generator.NumberGenerator"&gt;
        &lt;property name="nulls" value="0"/&gt;
      &lt;/generator&gt;
    &lt;/column&gt;
    &lt;column name="country_id" databaseDefault="false"&gt;
      &lt;generator type="pl.kernelpanic.dbmonster.generator.NumberGenerator"&gt;
        &lt;property name="nulls" value="0"/&gt;
      &lt;/generator&gt;
    &lt;/column&gt;
  &lt;/table&gt;
&lt;/dbmonster-schema&gt;
</pre>
<p>
The file is pretty self explanatory. However, here are some pointers:</p>
<ul>
<li>You have a table tag for every generated table</li>
<li>A column tag for every column of every table (duh)</li>
<li>Column's have generators. These are the Java classes that actually generate the value to insert into this column. Notice how you can specify the databasedefault="true" attribute, which makes dbmonster omit generation for this column (good for auto increment columns / postgres sequences</li>
</ul>
<p>
Now let's run it:</p>
<pre>
export CLASSPATH=dbmonster*.jar:mysql*jar

java pl.kernelpanic.dbmonster.Launcher -s sample_db.xml

fipar@telecaster:~/soft/dbmonster-core-1.0.3$ java -classpath mysql-connector-java-5.1.6-bin.jar:dbmonster-core-1.0.3.jar pl.kernelpanic.dbmonster.Launcher -s sample_db.xml
2009-08-05 13:43:27,244 INFO  DBMonster - Let's feed this hungry database.
2009-08-05 13:43:27,697 INFO  DBCPConnectionProvider - Today we are feeding: MySQL 5.0.75-0ubuntu10.2-log
2009-08-05 13:43:27,783 INFO  Schema - Generating schema <Sample DB>.
2009-08-05 13:43:27,783 INFO  Table - Generating table <tasks>.
2009-08-05 13:44:18,749 INFO  Table - Generation of table <tasks> finished.
2009-08-05 13:44:18,750 INFO  Schema - Generation of schema <Sample DB> finished.
2009-08-05 13:44:18,750 INFO  DBMonster - Finished in 51 sec. 507 ms.
fipar@telecaster:~/soft/dbmonster-core-1.0.3$ mysql -p -e 'select count(*) from tasks' sample_db
Enter password:
+----------+
| count(*) |
+----------+
|    80000 |
+----------+


fipar@telecaster:~/soft/dbmonster-core-1.0.3$ java -cp dbmonster-core-1.0.3.jar:mysql-connector-java-5.1.6-bin.jar pl.kernelpanic.dbmonster.Launcher -s sample_db.xml
2009-08-06 10:27:24,747 INFO  DBMonster - Let's feed this hungry database.
2009-08-06 10:27:25,336 INFO  DBCPConnectionProvider - Today we are feeding: MySQL 5.0.75-0ubuntu10.2-log
2009-08-06 10:27:25,490 INFO  Schema - Generating schema <Sample DB>.
2009-08-06 10:27:25,490 INFO  Table - Generating table <tasks>.
2009-08-06 10:37:14,284 INFO  Table - Generation of table <tasks> finished.
2009-08-06 10:37:14,284 INFO  Schema - Generation of schema <Sample DB> finished.
2009-08-06 10:37:14,285 INFO  DBMonster - Finished in 9 min. 49 sec. 539 ms.
fipar@telecaster:~/soft/dbmonster-core-1.0.3$ mysql -p -e 'select count(*) from tasks' sample_db
Enter password:
+----------+
| count(*) |
+----------+
|   150000 |
+----------+
</pre>
<p>
Besides the obvious difference in the number of rows, the second generation had larger text values (char(250) vs char(40)). You can see how that affected generation time. It should have finished  (proportionally) in<br />
about 1:50, yet it took dbmonser almost 10 minutes to generate this data.</p>
<p>
In conclusion, depending on the size of the database you need to generate, and the use you intend for it (forget testing a huge schema with referential constraints), dbmonster can certainly aid<br />
you in stress testing any database engine.</p>
