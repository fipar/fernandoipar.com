---
layout: post
title: Documenting emacs hyper shortcuts
redirect_from:
- 2020/04/29/documenting-emacs-hyper-shortcuts/ 
- 2020/04/29/documenting-emacs-hyper-shortcuts.html
image: /assets/images/shortcuts_documentation_sample.png

author: Fernando Ipar
status: publish
published: true
author_login: fernando
author_email:
categories:
- emacs

tags:
- notes
- emacs
---

I have written [before ](http://localhost:4000/emacs/notes/2018/01/02/emacs-hyper-key.html)about my discovery, and adoption of emacs shortcuts based on the Hyper key, and this time I want to share a simple script to generate basic documentation of them. 
The script works on the assumption that shortcuts that involve defining another shortcut (i.e., using kbd) will include some user-friendly documentation in the form of a comment, and will include this, instead of the kbd form, in the documentation. 
This is not perfect. For example, sometimes an extra closing parenthesis will show up. But perfect is the enemy of good, and this output is good enough for me right now. 

The color scheme is because I have physical sticker labels attached to some of my keys, and I am using [ and ] as 'map' keys to have two more layers of hyper shortcuts for each existing one (say, if 'H-f' focuses on the current org subtree, 'H-] f' reverts that, and I call the second form 'Hyper Map 1 f'), and for mnemonics I have colored [ green, and ] red, and I am using the colors on individual key labels to let me know what is there for each level. 

Why a pdf with the shortcuts if I have the labels, you may ask? Because while some keys are set in stone (I don't anticipate to ever change the shorcuts for getting me the day's agenda, or for invoking capture), others are more dynamic, and depend on how frequently I use (or stop using) a specific function. It's a pain to be relabeling these so I'm now just relying on the pdf document instead. 

Given the assumption described before, here's the script that gets me a pdf with my shortcuts (that looks like the one shown in the post's image): 

{% gist 618f12c996dd87d876d14f858254d9f2%}
