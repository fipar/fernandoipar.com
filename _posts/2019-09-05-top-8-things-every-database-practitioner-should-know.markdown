---
layout: post
redirect_from:
  - 2019/09/05/top-8-things-every-database-practitioner-should-know/
  - 2019/09/05/top-8-things-every-database-practitioner-should-know.html
image: /assets/images/millsap_book.jpg

author: Fernando Ipar
status: publish
published: true
title: Top 8 things every database practitioner should know 
author: Fernando Ipar
author_login: fernando
author_email:
categories:
- notes
tags:
- notes
- pythian
---

*This article is a cross post from [Pythian's blog](https://blog.pythian.com/the-top-8-things-every-database-practitioner-should-know/)*

I’ve been interviewing and mentoring candidates for technical database positions for almost ten years, and a common question from those who either don’t make it or feel they’re not ready yet to apply to the position they’re looking for is: What else should I learn to prepare for this?

This post is an attempt at summarizing my advice in this regard, covering what I think are the most important minimum topics one should be familiar with when applying for any position that involves doing technical work with databases. 

They are:

- Database Models
- Transactional concepts
- Data structures 
- Consensus algorithms
- Development and automation fundamentals
- Hardware basics
- Operating Systems Basics
- Performance Optimization

## Database Models

You need to start at the beginning, and when it comes to databases, I think that’s database models. A database model can be considered a specification for how data will look like to the users of the database, including which data types and operations are supported. Unless you’re developing a database system from scratch, you never choose a model directly; instead, you pick it by proxy when you decide which database system to use. 

For this reason, if your responsibilities include selecting a database based on a set of requirements, the only way to make an informed choice is by knowing which models are available, and which implementations they have. 

While studying the models, always keep in mind that, as practitioners, we’ll be making choices based not just on ideal models but also on real-world constraints (e.g., budget constraints), so more often than not we’ll find ourselves bending the rules. As in the case of making informed decisions, the only way to properly bend (or break) the rules is by first knowing them very well. 

I think the absolute minimum here is getting familiar with the relational model, as it inspired a lot of the database systems currently in use. You may have heard comments such as “when you get to a certain size, the relational model does not scale”. (A mistaken statement, since the relational model is a logical data model and has nothing to say about physical representation, so if anything, what does not scale is a specific implementation of the relational model. This statement makes as much sense as saying that arithmetic does not scale because your pocket calculator overflows at relatively small numbers, but I digress), but the fact is, there is still a vast market of job opportunities for people supporting database operations for medium-sized businesses, where a relational database implementation is a great fit. 

Next in line would be the most popular contemporary non-relational models. This is the world of object, document, graph, and time-series databases, among others. 

Finally, while not worth spending a lot of time (except to satisfy your historical curiosity), I think it pays to skim through pre-relational models, mostly so you become aware of 1) why the relational model became so popular, and 2) what pre-relational mistakes other contemporary products may be making. These would be the network and hierarchical models. 

Several textbooks can help with this, but my bare minimum recommendation is to read [“Database in Depth”](http://shop.oreilly.com/product/9780596100124.do) by C.J.Date. 

## Transactional Concepts

Another critical concept in database systems is transaction support. You should be familiar with the ACID acronym and be able to evaluate the transactional guarantees of a database system, as well as the transactional requirements of a system you may design or build. 

When using an SQL-based system, it’s essential to understand the transaction isolation levels provided by it. Make sure you go to the relevant documentation instead of relying just on the level names, as not all databases implement a given level in the same way. 

As mentioned, the first resource to learn this should be the official documentation for the product you’re using or evaluating. If you want to go deeper, [“Transactional Information Systems: Theory, Algorithms, and the Practice of Concurrency Control and Recovery”](https://dl.acm.org/citation.cfm?id=2821572) is an excellent resource. 

For distributed systems, [“Consistency Models”](https://jepsen.io/consistency) is a must-read. 

## Data Structures 

Data structures define the organization of data both in memory and in persistent storage. The fundamental ones you need to be familiar with to work with databases are B Trees, LSM Trees, and Hashes. 

Others that you may need to become familiar with depending on the products you use are Bitmap Indexes, Bloom Filters, and Sparse Indexes. 

Finally, if you want to get involved in developing or customizing database systems (e.g., patching MySQL, PostgreSQL, etc.) you should also be familiar with more fundamental data structures such as lists, stacks, and queues, among others. 

There are many sources of information about this, but I recommend 3: 

Some “Fundamentals” textbook. I learned with [Knuth](https://www-cs-faculty.stanford.edu/~knuth/taocp.html) (which, for data structures, involves Volumes [1](https://dl.acm.org/citation.cfm?id=260999) and [3](https://dl.acm.org/citation.cfm?id=280635)), but there are several excellent texts out there. Just check out a few samples on the web to find one where the style suits you and go for it. Online university course notes and bibliography are a great resource for finding out “where do I learn more?” too. 
The paper presenting a specific data structure, or a good write up of it. For example, here’s one for [LSM Trees](https://blog.acolyer.org/2014/11/26/the-log-structured-merge-tree-lsm-tree/) (I recommend The Morning Paper as a friendly way to keep up with our field). 
The relevant source code, if available. 
Finally, if you want to design your own data structure, I recommend [“The Periodic Table of Data Structures”](https://stratos.seas.harvard.edu/files/stratos/files/periodictabledatastructures.pdf) to get ideas of what you could try. 

## Consensus Algorithms

As soon as our databases involve more than one system communicating, they need some protocol for how to agree on what the global system’s state should be. 

Consensus algorithms are the theoretical foundation for this, and if you plan to work with distributed databases, you should become familiar (this means “can understand how it works and what guarantees are provided,” not “can implement it”) with some. The choice will be dictated by the system you use, but if you want to learn this before using a system, or, as in the case of learning database models, so you’re better informed when you have to choose which distributed system to use, I think it pays to become familiar with [Raft](https://raft.github.io/) as a starting point. [Paxos](https://lamport.azurewebsites.net/pubs/paxos-simple.pdf) can be an excellent second step, but bear in mind one of the key reasons why Raft was designed was to provide a consensus algorithm that was more understandable than Paxos, so be ready to spend some time if you want to go down this road (Notice I linked to “Paxos Made Simple” instead of to the [original paper](https://lamport.azurewebsites.net/pubs/lamport-paxos.pdf), even though they’re both by the same author). 

Another good piece of advice I can give here is, as practitioners, we should treat consensus algorithms like we (hopefully) treat cryptographic algorithms: you don’t make up your own. Don’t get me wrong: it’s perfectly fine to try to create something new (in fact, I encouraged you to design your own data structure just a few paragraphs above), but this is not something you “learn by doing,” much less by writing production software. 

## Development And Automation Fundamentals

The first thing you need to become proficient at is the language of the database you work with. The only caveat is that, even if you currently don’t use an SQL-based database, I think you should still become familiar with it since it is supported (in some form of another, with several inconsistencies between systems) by most database systems currently in use. As in the case of transaction isolation levels, I think the official documentation from the product should be read. However, that can be supplemented here by a more general SQL reference. My recommendation would be [“SQL and Relational Theory”](http://shop.oreilly.com/product/0636920046158.do) by C.J.Date. 

You should also be confident with scripting. Any scripting language available on the platform(s) you work should be good here. I’m a heavy bash user, but these days a lot of people go straight to Python for this. Unless the position you’re applying for lists a specific requirement, any language will do, since you’ll be able to apply the concepts from it into other scripting languages in the future if needed. 

Since database work involves a lot of automation these days, it’s essential to become familiar with some DevOps concepts and tools. At the very least, you’ll want to know the basics of automatic provisioning with [Ansible](https://www.ansible.com/), [Chef](https://www.chef.io/), or some other such tool. 

If you want to work on an Open Source database and you enjoy programming, it’s a great idea to become familiar with its source code, and, in turn, with the programming language it is written in. This may seem like overkill at first, but it is priceless when you need to troubleshoot hard-to-track problems or understand stack traces and other error messages. 


## Hardware Basics
You should be familiar with the overall architecture of the actual computers where the database software runs. 

Databases are meant (among other things) to provide a reliable and convenient abstraction for data storage so that users can just worry about their logic, and not care about how things are done under the hood. However, as the people supporting (I’m using the term broadly here) databases, we spend a significant part of our time under this hood. 

Among other things, this means you should be able to find out and properly interpret hardware specs such as storage device IOPS, understand what hyperthreading is, etc. All of this is tied together and, for example, what drives you have available may end up dictating which data structure you’ll use (e.g., some are better at reducing wear for SSDs or dealing with slower storage). 

There are several great books on computer architecture, but I think the best recommendation I can make for this and the next section is [“Systems Performance: Enterprise and The Cloud”](http://www.brendangregg.com/sysperfbook.html) by Brendan Gregg. 

## Operating Systems Basics

This is very much tied with Hardware and involves you knowing about swapping/paging, what Virtual Memory is, how the I/O subsystem works (including how to make sure a write made it to persistent storage, or what schedulers you have available). 

It’s also vital to know what tracing and monitoring tools your OS has available, so you can dig deeper when a performance problem requires it, and understand what resource is the bottleneck, for example. 

## Performance Optimization

Finally, you need to be proficient at suitable methods for performance analysis and optimization. Here, the Gregg book is recommended again, as it introduces the [USE method](http://www.brendangregg.com/usemethod.html), which I often use when diagnosing performance problems. 

The book [“Optimizing Oracle Performance”](http://shop.oreilly.com/product/9780596005276.do) by Cary Millsap is also recommended. If you don’t plan to work on Oracle: don’t be put off by the title. I’ve never worked on Oracle, and I still found it very helpful. I’d say about 2/3rds of the book is generic and applies to whatever database system you work with, as it consists of the presentation of “Method R” for performance optimization. 

As a final recommendation, for capacity planning, I recommend reading [“Guerrilla Capacity Planning”](https://www.springer.com/gp/book/9783540261384) by Neil Gunther. This book introduces the Universal Scalability Law and includes some heavy theoretical material (which I had to re-read several times) plus good practical examples for how to use it to estimate hardware and software scalability. 

## Parting words

I hope you found this useful. Besides studying all the things I’ve mentioned on this post, I have one last recommendation to make: get involved in an online community related (e.g., the [MySQL Community Slack](https://lefred.be/mysql-community-on-slack/)) to the technologies you want to get better at since frequently senior people are available and will provide insightful help to people asking questions. 
