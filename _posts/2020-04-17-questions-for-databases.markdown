---
layout: post
redirect_from:
  - 2020/04/17/questions-for-databases/
  - 2020/04/17/questions-for-databases.html
image: /assets/images/deadfish.jpg

author: Fernando Ipar
status: publish
published: true
title: Questions to ask when evaluating a database 
author: Fernando Ipar
author_login: fernando
author_email:
categories:
- notes
tags:
- notes
- databases
---

I've been evaluating a few databases recently while working on a way to process application logs, and here's a list of questions I ask myself during the process. 

This list is biased to operations, so I'm explicitly ignoring equally important questions that apply to the development and usage side, like "can it enforce user-defined integrity constraints on the data?". 

Without further ado, I want to know what happens when

- The process crashes, 
- The OS crashes, 
- It runs out of disk space, 
- It runs out of memory, 
- A system call (e.g., fork) fails,
- The system clock goes back (or forward) in time,
- The process is SIGSTOPped and SIGCONTed later,
- Multiple copies of the process are started with the same (or a different) config,
- The storage subsystem silently changes data on the write path,
- DNS fails,
- The network fails, or its performance degrades, or packets arrive out of order,
- The hostname changes,
- There's a surge in new connections, 
- A hardware resource saturates. 

More questions can be considered, (usually some will come up as subquestions while answering one of the above), but I've found this is a good enough list to start working on tests to verify how the database will behave for the use case. 

And what about the pic, you ask? It's a dead fish on the beach. Fish out of water. I want to know how the database dies, so to speak. Does it make an irrecoverable mess? Does it die as cleanly as possible and make an autopsy possible?
