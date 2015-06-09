---
layout: post
redirect_from:
  - 2015/06/08/org-mode-babel-thoughts/

status: publish
published: true
title: Some thoughts on org-mode/babel 
author:
  display_name: fernando
  login: fernando
  email:
  url: http://fernandoipar.com
author_login: fernando
author_email:
author_url: http://fernandoipar.com
categories:
- programming
tags:
- programming
- emacs
- org-mode
- babel
---

Lately I've been playing around with mongodb, specifically with the idea of having a script to summarize data that could be useful to have when troubleshooting, in a way like [Percona Toolkit's pt-mysql-summary](https://www.percona.com/doc/percona-toolkit/2.1/pt-mysql-summary.html) works for MySQL. 

Unrelated to this, I have been using [org-mode](http://orgmode.org/) to, well, organize my work, for a few years now. As part of my day to day job I write a lot of code snippets, usually in bash or SQL, sometimes in other languages, and for this I make heavy use of [code blocks](http://orgmode.org/manual/Working-With-Source-Code.html#Working-With-Source-Code). Over time, this has become one of my favourite org-mode's features, as it lets me keep code relevant to a customer issue on the same place as other information (like output from that code, output from text files, stack traces, notes from my calls with the customer, etc). 

Back to mongodb and my desire to work on some basic tools as a learning opportunity, I saw this as a good chance for trying to do some programming in the [literate](http://www.literateprogramming.com/) style. 

My first tests can be seen [here](https://github.com/fipar/playground/blob/master/tools-for-mongodb/mongo-summary.org). I understand completely this is a very small and simple project that is not the best to expose the potential benefits of this approach to programming, but it is what I can do now given my work/personal life obligations. It is my goal to start working on something bigger on the personal side, and I plan to use a similar approach to it, so I may know more after that. 

That said, one benefit I have already been able to appreciate is that literate programming with org-mode shines when your project combines different languages. If you look at the test functions, like the test for a sharded cluster of replica sets, you can see how org-mode lets me interweave bash and javascript as I explain what I am trying to achieve, in a way that's independent of how the code will be tangled (as each block will go to a file for the corresponding language). One benefit of this is improved readability (as the alternative would be to have cross-comments in the js and bash scripts, which I think would be less clear to follow), but another, potentially even bigger benefit, IMO, is that this makes it easier to combine different languages in a same project, which gives you more reasons to try and use the best tool for each job. If you don't need to generate a distributable 'executable' project (as is my goal here, I want people to be able to just run the generated code using the wrappers), then org-mode gives you even more features to combine different languages, by using result blocks associated with code blocks. I don't have a personal example for this right now, but you can read more about it [here](http://orgmode.org/manual/Evaluating-code-blocks.html#Evaluating-code-blocks). 