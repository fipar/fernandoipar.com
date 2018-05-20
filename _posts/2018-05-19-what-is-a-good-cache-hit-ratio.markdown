---
layout: post
redirect_from:
  - 2018/05/19/what-is-a-good-cache-hit-ratio/
  - 2018/05/19/what-is-a-good-cache-hit-ratio.html

author: Fernando Ipar
status: publish
published: true
title: What is a good cache hit ratio? 
author: Fernando Ipar
author_login: fernando
author_email:
categories:
- practice
tags:
- practice
- performance analysis
- databases
---

What is a good cache hit ratio?

Taking MySQL as an example: what is a good query cache hit ratio?

At the time of writing this, I get over 600 results if I enter that question into Google and limit the results to stackexchange.com. If I change 'ratio' to 'rate,' that increases to over 3000 results, which makes sense, as most people argue that it's the rate what you usually want to know about.

There are several answers to any of these questions, but I prefer to borrow from [Hofstadter's](https://en.wikipedia.org/wiki/Gödel,_Escher,_Bach) and [Pirsig's](https://en.wikipedia.org/wiki/Zen_and_the_Art_of_Motorcycle_Maintenance) interpretation of a Zen Koan and answer ['Mu'](https://en.wikipedia.org/wiki/Mu_(negative)#%22Unasking%22_the_question): unask the question.

What that means to me (and I am aware this is my interpretation of other people's interpretation of a concept in a culture foreign to them) is that it does not make sense to answer the question given the context. To be more specific, I don't think a (MySQL query) cache hit ratio can be good or bad. It can tell us something about a workload but nothing more. Additionally, what it tells us is aggregate information about this workload.

Let's say the hit rate is 60% hits over an hour. It does not tell us which queries were hits and which misses, nor what is the range of response time for a given query fingerprint (to borrow [pt-query-digest's](https://www.percona.com/doc/percona-toolkit/LATEST/pt-query-digest.html) term). Without this info, it does not tell us if the query cache is beneficial for the workload or not. The same applies for a hit rate of 90% hits over an hour, by the way, as explained by Peter Zaitsev for memcached in [this post](https://www.percona.com/blog/2010/05/19/beyond-great-cache-hit-ratio/). 

The reason I think this is not a valid question, at least not in a broad performance optimization/application health check context is that it tells us nothing about the things that actually matter about a database: is work being done at the rate and response time the client (or users) expect? Quoting [Cary Millsap](http://shop.oreilly.com/product/9780596005276.do), there are three only three acceptable units for (database) statistics that make sense to end users:

- your local currency

- The duration by which you’ll improve someone’s response time

- The number of business actions per unit of time by which you’ll improve someone’s throughput

A cache hit rate is measured in hits by time, which is none of the above because such a ratio tells you nothing useful about costs (or savings), response time or throughput.
