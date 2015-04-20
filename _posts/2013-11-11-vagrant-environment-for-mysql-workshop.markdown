---
layout: post
redirect_from:
  - 2013/11/11/vagrant-environment-for-mysql-workshop/

status: publish
published: true
title: Vagrant environment for MySQL Workshop
author:
  display_name: fernando
  login: fernando
  email: 
  url: http://fernandoipar.com
author_login: fernando
author_email: 
author_url: http://fernandoipar.com
wordpress_id: 272
wordpress_url: http://fernandoipar.com/?p=272
date: !binary |-
  MjAxMy0xMS0xMSAxNDoyMjoyOCAtMDIwMA==
date_gmt: !binary |-
  MjAxMy0xMS0xMSAxNzoyMjoyOCAtMDIwMA==
categories:
- Notes
tags: []
comments: []
---
<p>Repo: https://github.com/fipar/vagrant_mysql_workshop_2013</p>
<p>Base: https://github.com/fipar/vm_host_config/tree/master/vms/percona-server-5.6</p>
<p>Additional provisioning:</p>
<p><code> wget https://launchpad.net/test-db/employees-db-1/1.0.6/+download/employees_db-full-1.0.6.tar.bz2<br />
tar xjvf employees_db-full-1.0.6.tar.bz2 &amp;&amp; rm -f employees_db-full-1.0.6.tar.bz2<br />
cd employees_db<br />
mysql -t &lt; employees.sql<br />
wget http://downloads.mysql.com/docs/sakila-db.tar.gz<br />
tar xzvf sakila-db.tar.gz &amp;&amp; rm -f sakila-db.tar.gz<br />
mysql &lt; sakila-db/sakila-schema.sql<br />
mysql sakila &lt; sakila-db/sakila-data.sql<br />
</code></p>
