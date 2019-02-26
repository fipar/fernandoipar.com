---
layout: post
redirect_from:
  - 2018/01/02/emacs-hyper-key/
  - 2018/01/02/emacs-hyper-key.html
image: /assets/images/emacs-hyper-key.png

author: Fernando Ipar
status: publish
published: true
img: /assets/images/emacs-hyper-key.png
title: Expanding your keyboard on emacs with the Hyper key 
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

I recently stumbled upon the hyper and super keys on emacs <a href="http://ergoemacs.org/emacs/emacs_hyper_super_keys.html">here</a>. 

What I like about this setup is that I rarely use the Function key in my day to day usage, so this is making good use of a key that's both mostly unused and conveniently located in my keyboard, to the effect of running potentially complex tasks (like exporting the currently selected org mode subtree to a pdf and opening it) with just two keys. 

![Because every blog post needs a Featured Image](/assets/images/emacs-hyper-key.png)

Here's my current configuration:

	;; hyper key shortcuts: http://ergoemacs.org/emacs/emacs_hyper_super_keys.html
	(setq ns-function-modifier 'hyper)  ; make Fn key do Hyper

	(global-set-key (kbd "H-a") 'org-agenda)
	(global-set-key (kbd "H-d") 'delete-other-windows)
	(global-set-key (kbd "H-i") 'org-clock-in)
	(global-set-key (kbd "H-o") 'org-clock-out)
	(global-set-key (kbd "H-s") 'org-schedule)
	(global-set-key (kbd "H-b") 'ido-switch-buffer)
	(global-set-key (kbd "H-r") 'org-archive-subtree-default)
	(global-set-key (kbd "H-m") 'evil-mc-make-all-cursors)
	(global-set-key (kbd "H-n") 'evil-mc-undo-all-cursors)
	(global-set-key (kbd "H-k") 'epa-encrypt-region)
	(global-set-key (kbd "H-l") 'epa-decrypt-region)
	(global-set-key (kbd "H-j") 'eww)
	(global-set-key (kbd "H-v") 'darkroom-mode)
	(global-set-key (kbd "H-x") 'toggle-frame-fullscreen)


	(defun open-ansi-term-localhost ()
	(interactive)
	(ansi-term "/bin/bash" "localhost"))

	(global-set-key (kbd "H-t") 'open-ansi-term-localhost)
	;; C-c C-e C-s l o  export current subtree to latex pdf and open
	(global-set-key (kbd "H-e") (kbd "C-c C-e C-s l o"))
	;; same, but html buffer and open
	(global-set-key (kbd "H-w") (kbd "C-c C-e C-s h H"))
	;; same, but ascii and open
	(global-set-key (kbd "H-q") (kbd "C-c C-e C-s t A"))
	;; plain org export
	(global-set-key (kbd "H-y") 'org-export-dispatch) 
	;; org capture
	(global-set-key (kbd "H-c") 'org-capture) 
	;; eval lisp
	(global-set-key (kbd "H-z") 'org-ctrl-c-ctrl-c)



Something interesting here is that the target can be a function or another keyboard combination. The latter is useful if you can't easily find the function you want, or if you want to pass arguments along with it, like I am doing in my org export shortcuts.

As for how to keep up with the growing amount of hyper key based shortcuts, I am using the old school technology of little sticker labels with a descriptive text on each key. It's cheap and works wonders until I build the corresponding muscle memory. 

Nothing groundbreaking for seasoned emacs users but I thought it could be useful to newbies such as myself (I have been using the editor in one form or another for several years already, but with evil-mode, and using only a small subset of its features). 
