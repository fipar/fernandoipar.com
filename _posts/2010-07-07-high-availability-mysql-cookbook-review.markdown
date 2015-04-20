---
layout: post
redirect_from:
  - 2010/07/07/high-availability-mysql-cookbook-review/

status: publish
published: true
title: High Availability MySQL Cookbook review
author:
  display_name: fernando
  login: fernando
  email: 
  url: http://fernandoipar.com
author_login: fernando
author_email: 
author_url: http://fernandoipar.com
wordpress_id: 204
wordpress_url: http://fernandoipar.com/?p=204
date: !binary |-
  MjAxMC0wNy0wNyAwMTowMzoxNyAtMDMwMA==
date_gmt: !binary |-
  MjAxMC0wNy0wNyAwNDowMzoxNyAtMDMwMA==
categories:
- MySQL
tags:
- MySQL
- Availability
- Review
comments: []
---
<p><a href="https://www.packtpub.com/high-availability-mysql-cookbook/book?utm_source=fernandoipar.com&amp;utm_medium=bookrev&amp;utm_content=blog&amp;utm_campaign=mdb_003272">High Availability MySQL Cookbook</a> (Alex Davies, Packt Publishing) presents different approaches to achieve high availability with MySQL.</p>
<p>The bulk of the book is dedicated to MySQL Cluster, with shorter sections on:</p>
<ul>
<li>MySQL replication</li>
<li>shared storage</li>
<li>block level replication</li>
<li>performance tuning</li>
</ul>
<p>The recipes are clear and well explained, based on a CentOS distribution, and it seems any technically skilled person could follow them without issues.</p>
<p>What's lacking are some design aspects. Based on this material, one probably wouldn't be able to decide what the best high availability architecture is for a given problem. Actually, one may even be tempted to think MySQL Cluster is the best fit for most scenarios, given the percentage of the book dedicated to it. Nevertheless, there's a section about Cluster limitations and potential problems, so the cautious reader won't be tempted to choose this solution for every new project.</p>
<p>I also found that some important considerations regarding replication are missing.<br />
The reader is instructed to rely on Seconds_Behind_Master alone to monitor replication, and there's no mention to the situations that can cause as slave to go out of sync, nor of a process to fix this problem.</p>
<p>However, this book is a useful addition to any MySQL practitioner's library, provided you don't expect to rely only on it to design and deploy your MySQL based highly available services.</p>
