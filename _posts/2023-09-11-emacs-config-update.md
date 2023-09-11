---
layout: post
title: emacs config update
redirect_from:
- 2023/09/11/emacs-config-update/
- 2023/09/11/emacs-config-update.html
image: /assets/images/model-100.jpg

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

Time for the sporadic update about my emacs configuration journey.

I'm now on 29.1, and as part of the upgrade process, I refactored my config into more, smaller files, to make it easy to selectively disable features while troubelshooting startup problems.

Since I use both a personal and a work machine, maintaining emacs' configuration in sync has been a challenge. I've now gone back to keeping this on a private git repo, and I created a simple macro to support adding config snippets depending on the machine name:

{% gist ee28c09afd1a1e7bc3db0dfc94979b5c%}

For example, at work I'm using Github Copilot (through the neovim plugin and https://github.com/zerolfx/copilot.el) and I set this up on a file named copilot.el on my ~/.emacs.d/ with this content:

{% gist 816b8bf9f57d177da0ed28392b7baf39%}

I'm now simplifying my shortcuts a little bit by using [hydras](https://github.com/abo-abo/hydra), resulting in things like this:

{% gist 1a877880c3879b734e6715b7d4fb3b7a%}

Finally, I'm now using[ modus vivendi](https://github.com/protesilaos/modus-themes), which is now included on 29 (so I'm linking that repo just as a reference).
