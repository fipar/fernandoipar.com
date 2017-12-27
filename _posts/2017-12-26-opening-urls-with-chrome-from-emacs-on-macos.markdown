---
layout: post
redirect_from:
  - 2017/12/26/opening-urls-with-chrome-from-emacs-on-macos/
  - 2017/12/26/opening-urls-with-chrome-from-emacs-on-macos.html

author: Fernando Ipar
status: publish
published: true
title: Opening urls from emacs using Google Chrome on MacOS
author: Fernando Ipar
author_login: fernando
author_email:
categories:
- emacs
- notes
tags:
- emacs
- notes
---

I recently broke my emacs (aquamacs) setup. Fixing it was a very good learning experience that showed me how little I knew about how it actually worked!

As part of that process, I transitioned from aquamacs (a great project to which I made a small donation at some point) to just plain emacs. The big goal here is that since I spent most of my time in a text editor, I would like to be able to easily migrate myself to another platform supported by emacs with little need to adjust my habits (this has become an important requirement should I ever decide that Macs move too far away from the needs of users like me that I decide to switch back to FreeBSD or GNU/Linux).

I won't document the whole thing here, but I thought one little nugget of duck-tape-fixing was worth of making my blog-post-of-the-year on my personal blog, so here it is.

The scenario:

- I use Google Chrome for work as we have some customizer scripts that don't work in Safari.
- I do not want to make Chrome my default browser as I still use Safari for any browsing that does not involve my work email or my work's ticketing system.

My way to achieve this is by this emacs config line:

    (setq browse-url-browser-function 'browse-url-chromium)

That requires the 'chromium' command to be available on your path.
Now, at some point in the recent past, after a minor OS upgrade, emacs stopped being able to open URLs. After some digging, I tracked this down to the chromium command failing with this error:

    stratocaster:bin fipar$ chromium http://localhost
    dlopen /usr/local/bin/../Versions/65.0.3295.0/Chromium Framework.framework/Chromium Framework: dlopen(/usr/local/bin/../Versions/65.0.3295.0/Chromium Framework.framework/Chromium Framework, 261): image not found
    Abort trap: 6

I attempted reinstalling chromium (available on mac via homebrew or as a nightly build) but neither approach worked. After some thought, I realized I only need this so that emacs can open urls in Google, I don't use the chromium command for anything else. With that in mind, my duck tape solution was to create a chromium script somewhere on my PATH with the following contents:

    #!/bin/bash
    open -a 'Google Chrome' $*

That just open the standard MacOS way to open files with a specific application from the command line. Now I don't have the 'chromium' command installed, and I can open urls from emacs using Google Chrome just the same.

If you think this is a lazy approach to troubleshooting and that any self respected hacker would have instead spent time figuring out how to resolve the actual error, you're right. However, life is all about balancing needs and I just can't spend time tracking down how to fix a problem in a program I don't really need, so the lazy fix is enough for me at this point.

There's a lesson here for people like my younger past self: once you move into 'The Workforce', tech work is not about solving problems per se, but about adding to your client's revenue or cutting down from their costs. Doing this effectively requires, among other things, knowing when to stop because a solution is 'good enough' for the business need at hand, to the point where the benefits of continuing work on it no longer offset the costs involved.  

