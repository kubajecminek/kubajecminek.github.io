---
title: "Automating Gnus RSS Subscriptions"
date: "2023-12-13T19:03:04+02:00"
tags:
  - Emacs
---
[Gnus](https://www.gnu.org/software/emacs/manual/html_node/gnus/) is my
favourite newsreader; it can read e-mails, RSS and Atom feeds, tweets, Usenet
news and more. I was playing around with **nnrss** backend a lot this summer
and one thing that I did not particularly like was that you have to subscribe
to each RSS feed interactively (or at least this is what official
documentation recommends). Luckily this is Emacs so we can change just about
anything.

```emacs-lisp
(setq my-feeds '(("Richard Stallman" "https://stallman.org/rss/rss.xml" "Chief GNUisance of the GNU Project")
                ;; Define other feeds right here (title url description)
		 ))

;; This function mimics `gnus-group-make-rss-group' behaviour
(defun my-subscribe-rss-feeds (feeds)
  (require 'nnrss)
  (dolist (feed feeds)
    (let* ((title (nth 0 feed))
	   (href (nth 1 feed))
	   (desc (nth 2 feed))
	   (coding (gnus-group-name-charset '(nnrss "") title)))
      (when coding
	;; Unify non-ASCII text.
	(setq title (decode-coding-string
		     (encode-coding-string title coding)
		     coding)))
      (unless (gnus-group-entry (gnus-group-prefixed-name title "nnrss"))
	(gnus-group-make-group title '(nnrss ""))
	(push (list title href desc) nnrss-group-alist)
	(message "Group %s added" title)
	(nnrss-save-server-data nil)))))

(add-hook 'gnus-setup-news-hook (lambda () (my-subscribe-rss-feeds my-feeds)))
```

This snippet is also available on [Github](https://gist.github.com/kubajecminek/83cb82969acd9ec3e0a773a54cad2d13). Please let me know what is your Gnus/RSS setup.