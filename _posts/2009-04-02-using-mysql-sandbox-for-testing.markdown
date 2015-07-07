---
layout: post
redirect_from:
  - 2009/04/02/using-mysql-sandbox-for-testing/

author: Fernando Ipar
status: publish
published: true
title: Using MySQL sandbox for testing
author_login: fernando
author_email: 
wordpress_id: 136
date: !binary |-
  MjAwOS0wNC0wMiAyMToyODowNSAtMDMwMA==
date_gmt: !binary |-
  MjAwOS0wNC0wMiAyMzoyODowNSAtMDMwMA==
categories:
- MySQL
tags:
- MySQL
- Sandbox
- Testing
comments: []
---
<p><a title="MySQL Sandbox" href="https://launchpad.net/mysql-sandbox">MySQL Sandbox</a> is a great tool for quickly deploying test MySQL instances, particularly if your daily work involves diagnosing problems across multiple MySQL versions.</p>
<p>Once you've downloaded it, it needs no installation. Just have a few MySQL binary releases at hand, and begin creating sandboxes in just a few seconds:</p>
<blockquote><p>./make_sandbox mysql-5.0.77-linux-x86_64-glibc23.tar.gz</p></blockquote>
<p>or</p>
<blockquote><p>./make_sandbox mysql-5.1.32-linux-x86_64-glibc23.tar.gz</p></blockquote>
<p>Pretty simple, huh?</p>
<p>Suppose you have a parallel build around, say, 5.0.77-percona-highperf. The default syntax won't work if you've already created a 5.0.77 sandbox, since Sandbox will use '5.0.77' as the sandbox dir. In this case, you'll need to manually specify a directory name:</p>
<blockquote><p>./make_sandbox mysql-5.0.77-percona-highperf-b13.tar.gz -d msb_percona_5_0_77</p></blockquote>
<p>Fortunately, all Sandbox commands have a --help option that provide helpful screens to let you know how you can modify their default behavior. In this particular case, you'll actually need to access the help for low_level_make_sandbox, since that's the script that's eventually called.</p>
<p>Once a sandbox is created, using it is real simple. Just cd to it's directory (by default, $HOME/sandboxes/$SANDBOXDIR) and issue one of the following commands:</p>
<ul>
<li>./start</li>
<li>./stop</li>
<li>./restart</li>
<li>./use (this invokes the sandbox's mysql client with appropriate parameters. you can append your own parameters too, i.e., a database name)</li>
<li>./clear (stops the server and removes everything from the data directory, leaving you ready to start from scratch)</li>
</ul>
<p>It even includes an option to create replication sandboxes, with a master node and two slaves. This is by default, but you can also create multiple sandboxes of the same version and configure replication yourself.</p>
<p>Here's an example:</p>
<blockquote><p>./make_replication_sandbox mysql-5.0.77-linux-x86_64-glibc23.tar.gz</p></blockquote>
<p>So there it is, once again, <a title="MySQL Sandbox" href="https://launchpad.net/mysql-sandbox">download it</a> and start having fun!</p>
