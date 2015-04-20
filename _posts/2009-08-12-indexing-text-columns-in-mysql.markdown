---
layout: post
redirect_from:
  - 2009/08/12/indexing-text-columns-in-mysql/

status: publish
published: true
title: Indexing text columns in MySQL
author:
  display_name: fernando
  login: fernando
  email: 
  url: http://fernandoipar.com
author_login: fernando
author_email: 
author_url: http://fernandoipar.com
wordpress_id: 178
wordpress_url: http://fernandoipar.com/?p=178
date: !binary |-
  MjAwOS0wOC0xMiAyMDowNzoyOCAtMDMwMA==
date_gmt: !binary |-
  MjAwOS0wOC0xMiAyMjowNzoyOCAtMDMwMA==
categories:
- MySQL
tags:
- MySQL
- performance
- optimization
- indexes
comments:
- id: 70
  author: Using partial indexes in MySQL | Open Query blog
  author_email: ''
  author_url: http://openquery.com/blog/partial-indexes-string-columns
  date: !binary |-
    MjAwOS0wOC0xMyAyMjowMzo0MyAtMDMwMA==
  date_gmt: !binary |-
    MjAwOS0wOC0xNCAwMDowMzo0MyAtMDMwMA==
  content: ! '[...] reading Fernando Ipar&#8217;s interesting post on partial indexes
    for string columns, there were two things I wanted to [...]'
- id: 71
  author: Generating data with dbmonster | Fernando Ipar
  author_email: ''
  author_url: http://fernandoipar.com/2009/08/14/generating-data-with-dbmonster/
  date: !binary |-
    MjAwOS0wOC0xNCAxODowMzoxOSAtMDMwMA==
  date_gmt: !binary |-
    MjAwOS0wOC0xNCAyMDowMzoxOSAtMDMwMA==
  content: ! '[...] my last post I included some sample data which was useful for
    playing around with queries (once I published it, [...]'
- id: 109
  author: Julio
  author_email: info@velocidadmaxima.com
  author_url: http://VelocidadMaxima.com
  date: !binary |-
    MjAwOS0xMS0wOCAwNzo1NDo1MCAtMDIwMA==
  date_gmt: !binary |-
    MjAwOS0xMS0wOCAxMDo1NDo1MCAtMDIwMA==
  content: Thanks for this article, it was really helpful! =)
- id: 112
  author: JonB
  author_email: j_panadero@yahoo.com
  author_url: ''
  date: !binary |-
    MjAwOS0xMS0yNSAxOToyMDo1MCAtMDIwMA==
  date_gmt: !binary |-
    MjAwOS0xMS0yNSAyMjoyMDo1MCAtMDIwMA==
  content: Thanks for an insight into a very thorny problem I am tackling in my latest
    project. Now, I need to grok this and do some testing.
- id: 141
  author: rauni
  author_email: rmasta@gmail.com
  author_url: ''
  date: !binary |-
    MjAxMS0wMS0xOCAxMDo1ODoyNCAtMDIwMA==
  date_gmt: !binary |-
    MjAxMS0wMS0xOCAxMzo1ODoyNCAtMDIwMA==
  content: ! "It was very well written article - I liked that You showed the executed
    commands along with the tables, so readers can learn and try those commands on
    their own.\r\n\r\nThanks!"
- id: 142
  author: Indexing text &#8211; MySQL vs. MS SQL | DEEP in PHP
  author_email: ''
  author_url: http://www.deepinphp.com/2011/02/indexing-text-mysql-vs-ms-sql/
  date: !binary |-
    MjAxMS0wMi0wOCAxMjoxNzoxMyAtMDIwMA==
  date_gmt: !binary |-
    MjAxMS0wMi0wOCAxNToxNzoxMyAtMDIwMA==
  content: ! '[...] I know that MySQL have nice text indexes, which you have set on
    custom number of first characters, so i can balance it for the typical scenario
    (like this : http://fernandoipar.com/2009/08/12/indexing-text-columns-in-mysql/)
    [...]'
- id: 210
  author: Jason
  author_email: noemail@example.com
  author_url: ''
  date: !binary |-
    MjAxMy0xMS0wNCAxODowOToyOCAtMDIwMA==
  date_gmt: !binary |-
    MjAxMy0xMS0wNCAyMTowOToyOCAtMDIwMA==
  content: This has been extremely useful, thank you!
---
<p><P>This time, I'm talking about indexes for string typed columns. In particular, I'll show a procedure I find useful while looking for good index length values for these columns.<br />
</P><br />
<P>I'll use a sample table called people.<br />
</P><br />
<P>Here's what it looks like:<br />
</P><br />
<PRE>mysql&gt; desc people;<br />
+------------+------------------+------+-----+---------+----------------+<br />
| Field      | Type             | Null | Key | Default | Extra          |<br />
+------------+------------------+------+-----+---------+----------------+<br />
| id         | int(11) unsigned | NO   | PRI | NULL    | auto_increment |<br />
| title      | varchar(250)     | NO   |     | NULL    |                |<br />
| city       | varchar(250)     | NO   |     | NULL    |                |<br />
| occupation | varchar(250)     | NO   |     | NULL    |                |<br />
+------------+------------------+------+-----+---------+----------------+<br />
4 rows in set (0.00 sec)</p>
<p>mysql&gt; select count(*) from people;<br />
+----------+<br />
| count(*) |<br />
+----------+<br />
|   150000 |<br />
+----------+<br />
1 row in set (0.00 sec)</p>
<p>mysql&gt; </PRE><P><br />
We'll start by using procedure analyse to get some useful information<br />
about our data. Unless you know some fields are good candidates for<br />
use with the ENUM datatype, invoke procedure analyse with arguments<br />
(0,0) in order to prevent mysql from suggesting huge ENUMs for string<br />
columns.<br />
</P><br />
<PRE>mysql&gt; select * from people procedure analyse(0,0)\G<br />
*************************** 1. row ***************************<br />
             Field_name: test.people.id<br />
              Min_value: 1<br />
              Max_value: 150000<br />
             Min_length: 1<br />
             Max_length: 6<br />
       Empties_or_zeros: 0<br />
                  Nulls: 0<br />
Avg_value_or_avg_length: 75000.5000<br />
                    Std: 87258.1632<br />
      Optimal_fieldtype: MEDIUMINT(6) UNSIGNED NOT NULL<br />
*************************** 2. row ***************************<br />
             Field_name: test.people.title<br />
              Min_value: aback exclaims stopgap's chapel's tanked claps snowshoe cigarette correlates extras laster cluc<br />
              Max_value: Zulus colossally dictate cleft's enchanter del<br />
             Min_length: 40<br />
             Max_length: 150<br />
       Empties_or_zeros: 0<br />
                  Nulls: 0<br />
Avg_value_or_avg_length: 95.0869<br />
                    Std: NULL<br />
      Optimal_fieldtype: TINYTEXT NOT NULL<br />
*************************** 3. row ***************************<br />
             Field_name: test.people.city<br />
              Min_value: aback ascertaining unw<br />
              Max_value: Zulus imprisonments veiner a<br />
             Min_length: 5<br />
             Max_length: 30<br />
       Empties_or_zeros: 0<br />
                  Nulls: 0<br />
Avg_value_or_avg_length: 17.4861<br />
                    Std: NULL<br />
      Optimal_fieldtype: TINYTEXT NOT NULL<br />
*************************** 4. row ***************************<br />
             Field_name: test.people.occupation<br />
              Min_value:<br />
              Max_value:<br />
             Min_length: 0<br />
             Max_length: 0<br />
       Empties_or_zeros: 150000<br />
                  Nulls: 0<br />
Avg_value_or_avg_length: 0.0000<br />
                    Std: NULL<br />
      Optimal_fieldtype: CHAR(0) NOT NULL<br />
4 rows in set (0.19 sec)</p>
<p>mysql&gt; </PRE><P><br />
The id column is numeric, and is actually already indexed. We can't<br />
see this right here (though I showed an example of how you can use<br />
the output of procedure analyse and extend it to suit your needs, in<br />
which I did include an 'Indexed' column in the output), but we can<br />
see this, and gan more information from the table, with the following<br />
statement:<br />
</P><br />
<PRE>mysql&gt; show index from people;<br />
+--------+------------+----------+--------------+-------------+-----------+-------------+----------+--------+------+------------+---------+<br />
| Table  | Non_unique | Key_name | Seq_in_index | Column_name | Collation | Cardinality | Sub_part | Packed | Null | Index_type | Comment |<br />
+--------+------------+----------+--------------+-------------+-----------+-------------+----------+--------+------+------------+---------+<br />
| people |          0 | PRIMARY  |            1 | id          | A         |      150000 |     NULL | NULL   |      | BTREE      |         |<br />
+--------+------------+----------+--------------+-------------+-----------+-------------+----------+--------+------+------------+---------+<br />
1 row in set (0.00 sec)</p>
<p>mysql&gt;<br />
</PRE><P><br />
In order to create a good index for title, we can use the following query. I used 95 as a starting point, since it's reported by procedure analyse as the average data length of this column.<br />
</P></p>
<pre>
mysql> select count(distinct(substr(title,1,95))) / count(distinct(title)) * 100 from people;
+--------------------------------------------------------------------+
| count(distinct(substr(title,1,95))) / count(distinct(title)) * 100 |
+--------------------------------------------------------------------+
|                                                           100.0000 | 
+--------------------------------------------------------------------+
1 row in set (1.42 sec)

mysql> 

</pre>
<p><P><br />
As you can see, with 95 chars, we can get an index that covers 100% of the rows (i.e., get distinct values for all of them). Still, it's a big number. Using this query, we<br />
can begin to play a little bit with the index size, until we get to a good compromise between enough distinct values and an index that's small enough to be processed fast and<br />
maybe even loaded into memory.<br />
</P></p>
<pre>
mysql> select count(distinct(substr(title,1,20))) / count(distinct(title)) * 100 from people;
+--------------------------------------------------------------------+
| count(distinct(substr(title,1,20))) / count(distinct(title)) * 100 |
+--------------------------------------------------------------------+
|                                                            99.9507 | 
+--------------------------------------------------------------------+
1 row in set (1.15 sec)

mysql> 
</pre>
<p><P><br />
As it turns out, my data set doesn't require too many characters in order to be differentiated. Actually, I'm kind of cheating here, for a table this size, since I used a data generator to populate this tables, and it<br />
generated a lot of random text. Real world data would probably require a larger prefix in order to get such good differentiation. Anyway, let's push it a little bit more.<br />
</P></p>
<pre>
mysql> select count(distinct(substr(title,1,15))) / count(distinct(title)) * 100 from people;
+--------------------------------------------------------------------+
| count(distinct(substr(title,1,15))) / count(distinct(title)) * 100 |
+--------------------------------------------------------------------+
|                                                            97.0787 | 
+--------------------------------------------------------------------+
1 row in set (1.18 sec)

mysql> select count(distinct(substr(title,1,14))) / count(distinct(title)) * 100 from people;
+--------------------------------------------------------------------+
| count(distinct(substr(title,1,14))) / count(distinct(title)) * 100 |
+--------------------------------------------------------------------+
|                                                            94.4247 | 
+--------------------------------------------------------------------+
1 row in set (1.13 sec)
</pre>
<p><P><br />
Here's the turning point for me. A jumb between 94% and 97% of index coverage in just 1 character. So I'm sticking with 15. Let's test this with some queries.<br />
</P></p>
<pre>
mysql> select title from people limit 40;
+----------------------------------------------------------------------------------------------------------------------------------------------------+
| title                                                                                                                                              |
+----------------------------------------------------------------------------------------------------------------------------------------------------+
| puffs war's bruises buckles attainably Warnock's discoverer degeneration plots admirably assimilates germane burlesquely ri                        | 
| arbitrariness MacDraw's carbonates suckers budget chronicler cur drabs untested Aryans imperial                                                    | 
| commender dozes distills blackbird's mend meta                                                                                                     | 
| gallons haying occupation's sculpt fittingness scores onwards recessed masculineness denominator's regulated boyfriend's                           | 
| authored metaphor derivatively matchmakers ratification railing advantageousness flossing twin's barbarously infinite retreat alloying tenting t   | 
| Africans determinateness enquired quivers replaces nowhere applicability negative alarms lacquerer shivered arachnid ulcer sil                     | 
| filter offerings unboundedness clearness enthusiast commandants blunted betide rusticated blacks helmet's squabbles tasked Beethoven contro        | 
| thirties oftener tunnel anguish attainable formulat                                                                                                | 
| grotesquely fallacious inessential fain sanctioned too amplifi                                                                                     | 
| consort rapes deeply marker patterns compacted plumbe                                                                                              | 
| nasally combings searcher's pathname's bolts retrospective aroused squintingly boyish singers recompiles Austral                                   | 
| purpled draggingly nobody's luckier spinning goddess oscilloscopes aimer                                                                           | 
| aphasia reconverts shams entangle placer metaphysical visited turret nai                                                                           | 
| violation bituminous unweighed darkness cackles consonant foully fisted loci relishes burn m                                                       | 
| unsuffixed overdose humbles corpses fashions slashingly quietude delighte                                                                          | 
| sheller hypocrisy falser productions shied cube breed childishness requested pads redoub                                                           | 
| broils aorta refund sinker cankering reawakens portrayed resolving bard's stand ejects inhabitant's tittering genders proposition                  | 
| cyclone's glorification unrestricted delicately inhibitive waterway wardrobes excommunicated laugher                                               | 
| poppies heroine's gunner swollen reticle vertebrate's shrank unreliabilities infractions pretentious angstroms relations highness feasibil         | 
| ampoule clustering intermediaries honer ree                                                                                                        | 
| creature's transferals tidal unsigned stitching ought coerces visa girdling porn janitors parer song's croaked ta                                  | 
| hammer amalgamating stunting feasibility hopefulness oilier spraying frets pinks                                                                   | 
| comelier tomorrow's cowboy chalked lewdness cordial supering rut's neurally blindingly mute drowsiest gives in                                     | 
| slides aqueduct glazers abolition dangers sultry raid prominence hedges walks toppled defenders autocrat                                           | 
| theoretic thumps scum's photos bootlegged enveloper sallying populations disruptions inaugurate conclu                                             | 
| annotated bibliographies lichen user's bluebird's subproofs unendurably recollection's crumple                                                     | 
| sergeant outlets pinion reducer wiling impinge apes insaneness dose automatics lighthouse's cursory sleepily web's interruptions superin           | 
| tautened skylarks toad's seminar's archangel's sarcasm shipwrecks indeed incliner tying waterf                                                     | 
| chaotic censuses intimacy custodian's extendedly womb's safeguarding desire                                                                        | 
| abusiveness skippered inspirer enunciation taper memory's clearly guardianship inputed m                                                           | 
| firing anaphora subsegments turbulence affectedness refractory unsprayed chapter's volumes undramati                                               | 
| chef reception's glens budged budge arson assistance disagreeableness fodder garnering boated skater heroine's pamphle                             | 
| prophetic spilling asper petter's constable's classic ices teethes mails office's sordidness cylindered chaffing bivouac skeptics shuttering quash | 
| hoppers iciest sharer dietitian dictionaries frac                                                                                                  | 
| racketeer Ellen amounts origin's abstractions render vanish pantries retrieve Maxtor unprojected antithes                                          | 
| hinter arrangers dialogue imputing droppers shelver boyish demonstrator braving submitting operated carbonate protruding creasing prospecto        | 
| electrify garment linked discernible transceiver's ungrounded telegrapher uncoated                                                                 | 
| badly unaffected vex taming affiliation strings detracts grandpas girded cafeteria approving ideology froze underlinings assass                    | 
| MHz articulate draws transpires rubbling swarthier reeler bagged snug assisted consonant's settl                                                   | 
| taxi's wearying espies Anglican's intangibly fluent jugs liveth pride ex                                                                           | 
+----------------------------------------------------------------------------------------------------------------------------------------------------+
40 rows in set (0.00 sec)

mysql> 
</pre>
<p><P><br />
That should give you an idea of what type of needles we're looking for :)<br />
</P></p>
<pre>
mysql> select * from people where title like 'taxi%';
+--------+-----------------------------------------------------------------------------------------------------------------------------------------------------+--------------------------------+------------+
| id     | title                                                                                                                                               | city                           | occupation |
+--------+-----------------------------------------------------------------------------------------------------------------------------------------------------+--------------------------------+------------+
|     40 | taxi's wearying espies Anglican's intangibly fluent jugs liveth pride ex                                                                            | rotation tri                   |            | 
|   5736 | taxi's allegorically accounting manipulatory cautiousness computational promoter wool reproo                                                        | islan                          |            | 
|   6967 | taxis sprawls unblushing rude put absorbs reproducibilities crumblier kid                                                                           | DeMorgan overhe                |            | 
|   8388 | taxi catsup ornament transformer widener syndicates dismount pop t                                                                                  | exhibition's manages hedgehog' |            | 
|  10418 | taxingly eligibility whichever meditation corrosion unluckiness intoxicat                                                                           | gagged politeness looser       |            | 
|  13091 | taxi endowment watchfulness battalions stay trickle tangle blowfish maid's transmissions questionnaire vomit saner strokers constituent crab's      | populations window's zoo armie |            | 
|  16723 | taxi storing couldest bouts allegoric cluttered steeples fives hitchhike thrashes retirement de                                                     | envisaged maintain             |            | 
|  22168 | taxicab voicer controllers removing cellular houses router nourishing edict shrines strikes testicle's destine whale russeted certi                 | masturbates pu                 |            | 
|  24244 | taxing sleeve consultant's nonprogrammable twine delayer ingot respecter subex                                                                      | prematurely significant        |            | 
|  35595 | taxicabs aback spinal checkers germs overdraft's coon critter's patrician fled coalition massaging paced condemning impen                           | oppre                          |            | 
|  37105 | taxis articulatory indulgence bystanders skin burgess starlight calendaring aunt's bilging benightedness smallest softened xiv immerser fresher unn | crudely papally r              |            | 
|  40871 | taxicabs muzzling precocious resentment fellers pitiers beasts marines baselines diagrammatically clowning connecters stampedin                     | influencer                     |            | 
|  44298 | taxi rages unintelligibleness anastomosis orthogonally incompatibilities keypads hoarse province stamping perceived sh                              | unforgiving quiet              |            | 
|  46395 | taxi plunders novelty's downstairs newborn symbiotic climax highlights lounger keypads only schools possibilities                                   | flowing forgeries slende       |            | 
|  66078 | taxicab mercilessly excesses ships merchandising patch strobe                                                                                       | armfuls firmament hum coop     |            | 
|  71095 | taxing dispense regrettably resuspended kilobits downwardly domestically laps rainiest recapitulates despiser trophies chums a                      | enumerate indoctrinat          |            | 
|  77668 | taxicabs approachable disqualifying charcoaled script's o                                                                                           | kited publish disburse anarch  |            | 
|  84162 | taxicab's captivity dean eyeball uninspiring pawn's complication outcast's stared sneak s                                                           | impracticable dungeon crop     |            | 
|  87930 | taxi swiftly repacks unsupported slice mornings squares gland solar brainier harrying wag                                                           | cowslip halter plastics        |            | 
|  91282 | taxiing undetected cast commands clasping germina                                                                                                   | waxes her                      |            | 
| 104029 | taxied roofs besetting leadership electrocuted input metaphor bubbler vowing sponges assess                                                         | worthing understated bark      |            | 
| 105818 | taxis libretti defensively shoes antagonistically heavier endeared accidental gauging intercourse revolte                                           | runne                          |            | 
| 106163 | taxis bonfire's bench stereo preventer boringness blot's quieter acronyms transplant gained implores ba                                             | sighting leased sp             |            | 
| 122471 | taxicab's they've berries invader touching bumblingly courtier's boosting undisguised destroy amanuensis bangles digestiveness poppy's hulls        | purity professional unski      |            | 
| 123931 | taxicab's headgear Popek ratifying tenured Pascal's subduedly quitting earned planter forgave implicated noo                                        | bibliographies fraill          |            | 
| 127383 | taxi choir parameter's busted inspiration's fixated blinking complicator outwit plotters gobbles burningly leafed corruptively                      | radioed size telegr            |            | 
| 134211 | taxi's reconstructible indirect agglutination awaken eked unoccupied pillager subcomputation interviewing treader commending i                      | muddiness broom's              |            | 
| 145784 | taxicabs sanctuary armful battening terrifying impactors guns exchequer reigns laughter desolater s                                                 | buttonhole's isomorphism       |            | 
| 146371 | taxied carnivals giver misconceptions countenancer introduced anchovy exile pipelines weaned unabridged Britishly abyss's extenuating moodiness th  | penetrator upsho               |            | 
+--------+-----------------------------------------------------------------------------------------------------------------------------------------------------+--------------------------------+------------+
29 rows in set (0.11 sec)

mysql> explain select * from people where title like 'taxi%';
+----+-------------+--------+------+---------------+------+---------+------+--------+-------------+
| id | select_type | table  | type | possible_keys | key  | key_len | ref  | rows   | Extra       |
+----+-------------+--------+------+---------------+------+---------+------+--------+-------------+
|  1 | SIMPLE      | people | ALL  | NULL          | NULL | NULL    | NULL | 150000 | Using where | 
+----+-------------+--------+------+---------------+------+---------+------+--------+-------------+

mysql> select * from people where title = 'hoppers iciest sharer dietitian dictionaries frac';
+----+---------------------------------------------------+--------------------+------------+
| id | title                                             | city               | occupation |
+----+---------------------------------------------------+--------------------+------------+
| 34 | hoppers iciest sharer dietitian dictionaries frac | coroneted revolve  |            | 
+----+---------------------------------------------------+--------------------+------------+
1 row in set (0.12 sec)

mysql> explain select * from people where title = 'hoppers iciest sharer dietitian dictionaries frac';
+----+-------------+--------+------+---------------+------+---------+------+--------+-------------+
| id | select_type | table  | type | possible_keys | key  | key_len | ref  | rows   | Extra       |
+----+-------------+--------+------+---------------+------+---------+------+--------+-------------+
|  1 | SIMPLE      | people | ALL  | NULL          | NULL | NULL    | NULL | 150000 | Using where | 
+----+-------------+--------+------+---------------+------+---------+------+--------+-------------+
1 row in set (0.00 sec)
</pre>
<p>
Ok, now let's create the index. I'm loading it into a cache here, which is not necessary (even less given my size of 150000 tuples), but it helps. In order to do this, all the indexes in your table must have the same block size.</p>
<pre>
mysql> create index idx_people_title on people(title(15));
Query OK, 150000 rows affected (1.60 sec)
Records: 150000  Duplicates: 0  Warnings: 0

mysql> reset query cache;
Query OK, 0 rows affected (0.00 sec)

mysql> load index into cache people;
+-------------+--------------+----------+----------+
| Table       | Op           | Msg_type | Msg_text |
+-------------+--------------+----------+----------+
| test.people | preload_keys | status   | OK       | 
+-------------+--------------+----------+----------+
1 row in set (0.00 sec)

</pre>
<p>
Let's re test the queries:</p>
<pre>

mysql> select * from people where title like 'taxi%';
+--------+-----------------------------------------------------------------------------------------------------------------------------------------------------+--------------------------------+------------+
| id     | title                                                                                                                                               | city                           | occupation |
+--------+-----------------------------------------------------------------------------------------------------------------------------------------------------+--------------------------------+------------+
|   8388 | taxi catsup ornament transformer widener syndicates dismount pop t                                                                                  | exhibition's manages hedgehog' |            | 
| 127383 | taxi choir parameter's busted inspiration's fixated blinking complicator outwit plotters gobbles burningly leafed corruptively                      | radioed size telegr            |            | 
|  13091 | taxi endowment watchfulness battalions stay trickle tangle blowfish maid's transmissions questionnaire vomit saner strokers constituent crab's      | populations window's zoo armie |            | 
|  46395 | taxi plunders novelty's downstairs newborn symbiotic climax highlights lounger keypads only schools possibilities                                   | flowing forgeries slende       |            | 
|  44298 | taxi rages unintelligibleness anastomosis orthogonally incompatibilities keypads hoarse province stamping perceived sh                              | unforgiving quiet              |            | 
|  16723 | taxi storing couldest bouts allegoric cluttered steeples fives hitchhike thrashes retirement de                                                     | envisaged maintain             |            | 
|  87930 | taxi swiftly repacks unsupported slice mornings squares gland solar brainier harrying wag                                                           | cowslip halter plastics        |            | 
|   5736 | taxi's allegorically accounting manipulatory cautiousness computational promoter wool reproo                                                        | islan                          |            | 
| 134211 | taxi's reconstructible indirect agglutination awaken eked unoccupied pillager subcomputation interviewing treader commending i                      | muddiness broom's              |            | 
|     40 | taxi's wearying espies Anglican's intangibly fluent jugs liveth pride ex                                                                            | rotation tri                   |            | 
|  66078 | taxicab mercilessly excesses ships merchandising patch strobe                                                                                       | armfuls firmament hum coop     |            | 
|  22168 | taxicab voicer controllers removing cellular houses router nourishing edict shrines strikes testicle's destine whale russeted certi                 | masturbates pu                 |            | 
|  84162 | taxicab's captivity dean eyeball uninspiring pawn's complication outcast's stared sneak s                                                           | impracticable dungeon crop     |            | 
| 123931 | taxicab's headgear Popek ratifying tenured Pascal's subduedly quitting earned planter forgave implicated noo                                        | bibliographies fraill          |            | 
| 122471 | taxicab's they've berries invader touching bumblingly courtier's boosting undisguised destroy amanuensis bangles digestiveness poppy's hulls        | purity professional unski      |            | 
|  35595 | taxicabs aback spinal checkers germs overdraft's coon critter's patrician fled coalition massaging paced condemning impen                           | oppre                          |            | 
|  77668 | taxicabs approachable disqualifying charcoaled script's o                                                                                           | kited publish disburse anarch  |            | 
|  40871 | taxicabs muzzling precocious resentment fellers pitiers beasts marines baselines diagrammatically clowning connecters stampedin                     | influencer                     |            | 
| 145784 | taxicabs sanctuary armful battening terrifying impactors guns exchequer reigns laughter desolater s                                                 | buttonhole's isomorphism       |            | 
| 146371 | taxied carnivals giver misconceptions countenancer introduced anchovy exile pipelines weaned unabridged Britishly abyss's extenuating moodiness th  | penetrator upsho               |            | 
| 104029 | taxied roofs besetting leadership electrocuted input metaphor bubbler vowing sponges assess                                                         | worthing understated bark      |            | 
|  91282 | taxiing undetected cast commands clasping germina                                                                                                   | waxes her                      |            | 
|  71095 | taxing dispense regrettably resuspended kilobits downwardly domestically laps rainiest recapitulates despiser trophies chums a                      | enumerate indoctrinat          |            | 
|  24244 | taxing sleeve consultant's nonprogrammable twine delayer ingot respecter subex                                                                      | prematurely significant        |            | 
|  10418 | taxingly eligibility whichever meditation corrosion unluckiness intoxicat                                                                           | gagged politeness looser       |            | 
|  37105 | taxis articulatory indulgence bystanders skin burgess starlight calendaring aunt's bilging benightedness smallest softened xiv immerser fresher unn | crudely papally r              |            | 
| 106163 | taxis bonfire's bench stereo preventer boringness blot's quieter acronyms transplant gained implores ba                                             | sighting leased sp             |            | 
| 105818 | taxis libretti defensively shoes antagonistically heavier endeared accidental gauging intercourse revolte                                           | runne                          |            | 
|   6967 | taxis sprawls unblushing rude put absorbs reproducibilities crumblier kid                                                                           | DeMorgan overhe                |            | 
+--------+-----------------------------------------------------------------------------------------------------------------------------------------------------+--------------------------------+------------+
29 rows in set (0.00 sec)

mysql> explain select * from people where title like 'taxi%';
+----+-------------+--------+-------+------------------+------------------+---------+------+------+-------------+
| id | select_type | table  | type  | possible_keys    | key              | key_len | ref  | rows | Extra       |
+----+-------------+--------+-------+------------------+------------------+---------+------+------+-------------+
|  1 | SIMPLE      | people | range | idx_people_title | idx_people_title | 17      | NULL |   56 | Using where | 
+----+-------------+--------+-------+------------------+------------------+---------+------+------+-------------+
1 row in set (0.00 sec)
</pre>
<p>
Notice how we only go through 56 rows now.</p>
<pre>

mysql> select * from people where title = 'hoppers iciest sharer dietitian dictionaries frac';
+----+---------------------------------------------------+--------------------+------------+
| id | title                                             | city               | occupation |
+----+---------------------------------------------------+--------------------+------------+
| 34 | hoppers iciest sharer dietitian dictionaries frac | coroneted revolve  |            | 
+----+---------------------------------------------------+--------------------+------------+
1 row in set (0.00 sec)

mysql> explain select * from people where title = 'hoppers iciest sharer dietitian dictionaries frac';
+----+-------------+--------+------+------------------+------------------+---------+-------+------+-------------+
| id | select_type | table  | type | possible_keys    | key              | key_len | ref   | rows | Extra       |
+----+-------------+--------+------+------------------+------------------+---------+-------+------+-------------+
|  1 | SIMPLE      | people | ref  | idx_people_title | idx_people_title | 17      | const |    1 | Using where | 
+----+-------------+--------+------+------------------+------------------+---------+-------+------+-------------+
1 row in set (0.01 sec)
</pre>
<p>
Just 1 row.<br />
Ok, let's test the quality of the index to find unique rows.</p>
<pre>

mysql> explain select * from people where title = 'arbitrariness MacDraw
<p>
Granted, 150.000 rows isn't much, but still, with an average row data length of 94, I had to find a 140 character title in order to go through 2 rows before the right one was found. That's reasonable, considering<br />
I estimated a 97% index coverage.</p>
<p>
In conclusion, wihle my dataset size certainly isn't large enough to do many interesting things, it should prove the point that a good index size will go great lengths into helping you improve the performance of your MySQL based<br />
system. The query I presented here can be useful to look for a decent index size in terms of unique rows coverage.</p>
s carbonates suckers budget chronicler cur drabs untested Aryans imperial';
+----+-------------+--------+------+------------------+------------------+---------+-------+------+-------------+
| id | select_type | table  | type | possible_keys    | key              | key_len | ref   | rows | Extra       |
+----+-------------+--------+------+------------------+------------------+---------+-------+------+-------------+
|  1 | SIMPLE      | people | ref  | idx_people_title | idx_people_title | 17      | const |    1 | Using where | 
+----+-------------+--------+------+------------------+------------------+---------+-------+------+-------------+
1 row in set (0.00 sec)

mysql> explain select * from people where title = 'dunes delightfulness manurers jousts axer aristocrat
<p>
Granted, 150.000 rows isn't much, but still, with an average row data length of 94, I had to find a 140 character title in order to go through 2 rows before the right one was found. That's reasonable, considering<br />
I estimated a 97% index coverage.</p>
<p>
In conclusion, wihle my dataset size certainly isn't large enough to do many interesting things, it should prove the point that a good index size will go great lengths into helping you improve the performance of your MySQL based<br />
system. The query I presented here can be useful to look for a decent index size in terms of unique rows coverage.</p>
s driver greediness bloke pays preconditions enclosure consideration plaster';
+----+-------------+--------+------+------------------+------------------+---------+-------+------+-------------+
| id | select_type | table  | type | possible_keys    | key              | key_len | ref   | rows | Extra       |
+----+-------------+--------+------+------------------+------------------+---------+-------+------+-------------+
|  1 | SIMPLE      | people | ref  | idx_people_title | idx_people_title | 17      | const |    1 | Using where | 
+----+-------------+--------+------+------------------+------------------+---------+-------+------+-------------+
1 row in set (0.00 sec)

mysql> explain select * from people where title = 'satire
<p>
Granted, 150.000 rows isn't much, but still, with an average row data length of 94, I had to find a 140 character title in order to go through 2 rows before the right one was found. That's reasonable, considering<br />
I estimated a 97% index coverage.</p>
<p>
In conclusion, wihle my dataset size certainly isn't large enough to do many interesting things, it should prove the point that a good index size will go great lengths into helping you improve the performance of your MySQL based<br />
system. The query I presented here can be useful to look for a decent index size in terms of unique rows coverage.</p>
s most quacked campaigning wrists disengaging insignia woodlander knuckles despaired portending incredulous predication Sally
<p>
Granted, 150.000 rows isn't much, but still, with an average row data length of 94, I had to find a 140 character title in order to go through 2 rows before the right one was found. That's reasonable, considering<br />
I estimated a 97% index coverage.</p>
<p>
In conclusion, wihle my dataset size certainly isn't large enough to do many interesting things, it should prove the point that a good index size will go great lengths into helping you improve the performance of your MySQL based<br />
system. The query I presented here can be useful to look for a decent index size in terms of unique rows coverage.</p>
s amica';
+----+-------------+--------+------+------------------+------------------+---------+-------+------+-------------+
| id | select_type | table  | type | possible_keys    | key              | key_len | ref   | rows | Extra       |
+----+-------------+--------+------+------------------+------------------+---------+-------+------+-------------+
|  1 | SIMPLE      | people | ref  | idx_people_title | idx_people_title | 17      | const |    2 | Using where | 
+----+-------------+--------+------+------------------+------------------+---------+-------+------+-------------+
1 row in set (0.00 sec)

mysql> select *,length(title) from people where title = 'satire
<p>
Granted, 150.000 rows isn't much, but still, with an average row data length of 94, I had to find a 140 character title in order to go through 2 rows before the right one was found. That's reasonable, considering<br />
I estimated a 97% index coverage.</p>
<p>
In conclusion, wihle my dataset size certainly isn't large enough to do many interesting things, it should prove the point that a good index size will go great lengths into helping you improve the performance of your MySQL based<br />
system. The query I presented here can be useful to look for a decent index size in terms of unique rows coverage.</p>
s most quacked campaigning wrists disengaging insignia woodlander knuckles despaired portending incredulous predication Sally
<p>
Granted, 150.000 rows isn't much, but still, with an average row data length of 94, I had to find a 140 character title in order to go through 2 rows before the right one was found. That's reasonable, considering<br />
I estimated a 97% index coverage.</p>
<p>
In conclusion, wihle my dataset size certainly isn't large enough to do many interesting things, it should prove the point that a good index size will go great lengths into helping you improve the performance of your MySQL based<br />
system. The query I presented here can be useful to look for a decent index size in terms of unique rows coverage.</p>
s amica';
+-----+----------------------------------------------------------------------------------------------------------------------------------------------+------------------+------------+---------------+
| id  | title                                                                                                                                        | city             | occupation | length(title) |
+-----+----------------------------------------------------------------------------------------------------------------------------------------------+------------------+------------+---------------+
| 344 | satire's most quacked campaigning wrists disengaging insignia woodlander knuckles despaired portending incredulous predication Sally's amica | buffaloes refill |            |           140 | 
+-----+----------------------------------------------------------------------------------------------------------------------------------------------+------------------+------------+---------------+
1 row in set (0.00 sec)

</pre>
<p>
Granted, 150.000 rows isn't much, but still, with an average row data length of 94, I had to find a 140 character title in order to go through 2 rows before the right one was found. That's reasonable, considering<br />
I estimated a 97% index coverage.</p>
<p>
In conclusion, wihle my dataset size certainly isn't large enough to do many interesting things, it should prove the point that a good index size will go great lengths into helping you improve the performance of your MySQL based<br />
system. The query I presented here can be useful to look for a decent index size in terms of unique rows coverage.</p>
