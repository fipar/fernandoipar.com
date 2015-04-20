---
layout: post
redirect_from:
  - 2013/11/19/rmysql-basics/

status: publish
published: true
title: RMySQL basics
author:
  display_name: fernando
  login: fernando
  email: 
  url: http://fernandoipar.com
author_login: fernando
author_email: 
author_url: http://fernandoipar.com
wordpress_id: 289
wordpress_url: http://fernandoipar.com/?p=289
date: !binary |-
  MjAxMy0xMS0xOSAxNzozMToyMyAtMDIwMA==
date_gmt: !binary |-
  MjAxMy0xMS0xOSAyMDozMToyMyAtMDIwMA==
categories:
- Notes
tags:
- MySQL
- R
comments:
- id: 239
  author: fernando
  author_email: 
  author_url: http://fernandoipar.com
  date: !binary |-
    MjAxMy0xMi0wMiAxOToyNToyOCAtMDIwMA==
  date_gmt: !binary |-
    MjAxMy0xMi0wMiAyMjoyNToyOCAtMDIwMA==
  content: ! "More detailed example w/graph: \r\n\r\nhttps://gist.github.com/fipar/7758814"
---
<p>Installation:<br />
<code><br />
> install.packages("RMySQL", dependencies = TRUE)<br />
> library(RMySQL)<br />
</code></p>
<p>Simple usage:<br />
<code><br />
> con <- dbConnect(MySQL(), user="msandbox", password="msandbox", dbname="sakila", host="127.0.0.1", port=5527)<br />
> dbGetQuery(con, "select * from sakila.sales_by_store")<br />
                store      manager total_sales<br />
1 Woodridge,Australia Jon Stephens    33726.77<br />
2   Lethbridge,Canada Mike Hillyer    33679.79<br />
> sales_by_film_category <- dbReadTable(con, "sales_by_film_category")<br />
> qplot(category, total_sales, data=sales_by_film_category, geom="bar", fill=category)</p>
<p></code></p>
