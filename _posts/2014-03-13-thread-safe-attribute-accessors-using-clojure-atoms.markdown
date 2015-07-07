---
layout: post
redirect_from:
  - 2014/03/13/thread-safe-attribute-accessors-using-clojure-atoms/

author: Fernando Ipar
status: publish
published: true
title: thread safe attribute accessors for ruby classes, using clojure Atoms
author_login: fernando
author_email: 
wordpress_id: 318
date: !binary |-
  MjAxNC0wMy0xMyAyMjowMjozMCAtMDMwMA==
date_gmt: !binary |-
  MjAxNC0wMy0xNCAwMTowMjozMCAtMDMwMA==
categories:
- Programming
tags:
- Programming
- concurrency
- Java
- Ruby
- Clojure
comments:
- id: 276
  author: thread safe accessors for ruby classes using clojure atoms, now packaged
    as gem |
  author_email: ''
  date: !binary |-
    MjAxNC0wMy0xNyAyMDoxMTozMCAtMDMwMA==
  date_gmt: !binary |-
    MjAxNC0wMy0xNyAyMzoxMTozMCAtMDMwMA==
  content: ! '[&#8230;] mentioned this before, and now I have packaged this as a (j)ruby
    gem for ease of [&#8230;]'
---
<p>This <a href="https://gist.github.com/fipar/9539935">code snippet</a> shows how to use Clojure Atoms to implement thread safe instance variables in JRuby classes.</p>
<p>The atom_attr_accessor method is used analog to the usual attr_accessor so this should seem natural for the language, though I don't have enough experience with it to judge that.</p>
