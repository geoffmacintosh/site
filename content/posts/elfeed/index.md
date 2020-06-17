+++
title = "Elfeed"
author = ["Geoff MacIntosh"]
date = 2020-05-20T00:00:00-02:30
tags = ["emacs"]
draft = false
+++

## Introduction {#introduction}

Usually people start these things out by explaining what RSS is and all that. I don't think I'll be doing that. I like RSS because I like knowing when new things happen, and I don't want to check a bunch of different services all the time. Beyond that, I also really like the idea of being able to filter out feed items that don't appeal to me. I don't mind if I can only read stuff on my computer, so I haven't set up any sort of sync with my phone, although it should be possible to do that.

I have [Elfeed](https://github.com/skeeto/elfeed) set up in a single use-package declaration, and I've pulled all the individual functions out into their own bits so as to talk about them separately.

```emacs-lisp
(use-package elfeed
  :straight t
  :bind
  (("C-x w" . actuator-elfeed-load-db-and-open)
   :map elfeed-search-mode-map
   ("A" . actuator-elfeed-show-all)
   ("U" . actuator-elfeed-show-unread)
   ("q" . actuator-elfeed-save-db-and-bury)
   ("R" . actuator-elfeed-mark-all-as-read))
  :custom
  (elfeed-search-filter "@1-week-ago +unread ")
  :config
  <<shortcuts>>
  <<faces>>
  <<filters>>
  <<feeds>>
  <<load-quit>>
  <<mark-all-as-read>>)
```


## Shortcuts {#shortcuts}

I built a few shortcuts to switch between different tag views that I commonly use. Elfeed has support for Emacs' bookmarks, so I just needed to make bookmarks for the views I wanted. I set up the search how I like it (`s`) then made a bookmark entry (`C-x r m`) called, say `elfeed-all`. I can call that bookmark from anywhere in Emacs to go to that elfeed view, but I also decided to [steal some functions from Pragmatic Emacs](http://pragmaticemacs.com/emacs/read-your-rss-feeds-in-emacs-with-elfeed/) to make single-letter keybindings in elfeed.

```emacs-lisp
(defun actuator-elfeed-show-all ()
  (interactive)
  (bookmark-maybe-load-default-file)
  (bookmark-jump "elfeed-all"))
(defun actuator-elfeed-show-unread ()
  (interactive)
  (bookmark-maybe-load-default-file)
  (bookmark-jump "elfeed-unread"))
```


## Filters {#filters}

Filters are kind of the star of Elfeed. I mostly use them to remove items that I don't want to see (or already see in other contexts---podcasts for example). I think it's all pretty straightforward. The only thing of note that I do is adding a debug tag to each hook that hides things. That way I can tell which filter it is that's causing problems when I make a stupid typo and suddenly a specific filter matches all entries.

```emacs-lisp
(add-hook 'elfeed-new-entry-hook
          (elfeed-make-tagger :entry-title "sponsor\\|revenue\\|financial"
                              :add '(junk debug1)
                              :remove 'unread))
(add-hook 'elfeed-new-entry-hook
          (elfeed-make-tagger :before "2 weeks ago"
                              :add 'debug2
                              :remove 'unread))
(add-hook 'elfeed-new-entry-hook
          (elfeed-make-tagger :feed-title "MacSparky"
                              :entry-title "focused\\|Mac Power Users\\|jazz\\|automators\\|podcast"
                              :add '(junk debug3)
                              :remove 'unread))
(add-hook 'elfeed-new-entry-hook
          (elfeed-make-tagger :feed-title "Six Colors"
                              :entry-title "podcast\\|macworld"
                              :add '(junk debug4)
                              :remove 'unread))
(add-hook 'elfeed-new-entry-hook
          (elfeed-make-tagger :feed-title "Longreads"
                              :entry-title "longreads"
                              :add '(junk debug5)
                              :remove 'unread))
(add-hook 'elfeed-new-entry-hook
          (elfeed-make-tagger :feed-url "youtube\\.com"
                              :add '(video youtube)))
(add-hook 'elfeed-new-entry-hook
          (elfeed-make-tagger :feed-url "twitchrss"
                              :add '(video twitch)))
(add-hook 'elfeed-new-entry-hook
          (elfeed-make-tagger :feed-url "kijiji\\.ca"
                              :add '(shop kijiji)))
(add-hook 'elfeed-new-entry-hook
          (elfeed-make-tagger :feed-url "reddit"
                              :add 'reddit))
(add-hook 'elfeed-new-entry-hook
          (elfeed-make-tagger :feed-url "ikea"
                              :entry-title "Q\\:"
                              :remove 'unread
                              :add '(junk debug6)))
(add-hook 'elfeed-new-entry-hook
          (elfeed-make-tagger :feed-url "cestlaz"
                              :entry-title '(not "emacs")
                              :add '(junk debug7)
                              :remove 'unread))
```


## Load and quit Elfeed nicely {#load-and-quit-elfeed-nicely}

You don't need to do anything special to load Elfeed. You can set up a keybinding that runs `(elfeed)` and it should work. I took this function from [Pragmatic Emacs](http://pragmaticemacs.com/emacs/read-your-rss-feeds-in-emacs-with-elfeed/) when I first set up Elfeed a few years ago because I wanted to keep the database in sync between multiple computers. These helper functions ensure that the database is loaded and saved at the appropriate moments. I'm not sure there's any benefit to these if you only use them on one computer (as I do now) but I can't find any downsides either, so they stay.

```emacs-lisp
(defun actuator-elfeed-load-db-and-open ()
      "Wrapper to load the elfeed database from disk before
      opening. Taken from Pragmatic Emacs."
      (interactive)
      (window-configuration-to-register :elfeed-fullscreen)
      (delete-other-windows)
      (elfeed)
      (elfeed-db-load)
      (elfeed-search-update 1)
      (elfeed-update))
```

```emacs-lisp
(defun actuator-elfeed-save-db-and-bury ()
  "Wrapper to save the Elfeed database to disk before burying
  buffer. Taken from Pragmatic Emacs."
  (interactive)
  (elfeed-db-save)
  (quit-window)
  (garbage-collect)
  (jump-to-register :elfeed-fullscreen))
```


## Mark all as read {#mark-all-as-read}

You can just go post-by-post and use `r` to mark individual posts as read. I stole this function from [Mike Zamansky](https://cestlaz-nikola.github.io/posts/using-emacs-29%20elfeed/) because it seemed like a nice addition.

```emacs-lisp
(defun actuator-elfeed-mark-all-as-read ()
    "Mark all feeds in search as read. Taken from Mike Zamansky"
    (interactive)
    (mark-whole-buffer)
    (elfeed-search-untag-all-unread))
```


## Faces {#faces}

Changing the colours of an entry is neat, but not that useful. I mostly have this set up in order to learn how to do it, and as a vague novelty.

```emacs-lisp
(add-to-list 'elfeed-search-face-alist
             '(video actuator-elfeed-video-face))
(add-to-list 'elfeed-search-face-alist
             '(image actuator-elfeed-image-face))
(add-to-list 'elfeed-search-face-alist
             '(comic actuator-elfeed-comic-face))
```

```emacs-lisp
(defface actuator-elfeed-video-face
  `((t . (:background "gray90" :foreground "blue")))
  "Face for elfeed video entry."
  :group 'actuator-elfeed)
```

```emacs-lisp
(defface actuator-elfeed-image-face
  `((t . (:background "gray90" :foreground "blue")))
  "Face for elfeed image entry."
  :group 'actuator-elfeed)
```

```emacs-lisp
(defface actuator-elfeed-comic-face
  `((t . (:background "gray90" :foreground "blue")))
  "Face for elfeed comic entry."
  :group 'actuator-elfeed)
```


## Feeds {#feeds}

I'm actually surprised I don't use the excellent [Elfeed-org](https://github.com/remyhonig/elfeed-org) package. I have used it in the past, but I don't anymore. I don't like Org-mode documents where headlines are also links, and I don't value having much of a hiearchy for tags. I keep considering setting it up just so I can nicely rename all my feeds to be consistent, but I just haven't bothered.

```emacs-lisp
(setq elfeed-feeds
      '(("https://www.youtube.com/feeds/videos.xml?channel_id=UCwBbuLWaIhxGuA6THzAqqIQ")
        ("http://approachingpavonis.blogspot.com/feeds/posts/default")
        ("https://www.youtube.com/feeds/videos.xml?channel_id=UCVdQKW6fmfBmhz4t5k8Dq5w")
        ("https://www.youtube.com/feeds/videos.xml?channel_id=UCkcODH4P9o3ovGWCRV5kJkA")
        ("https://www.youtube.com/feeds/videos.xml?channel_id=UC8tThli1ZY7LW5Dxqr3Y0jA")
        ("https://www.youtube.com/feeds/videos.xml?channel_id=UCbpMy0Fg74eXXkvxJrtEn3w")
        ("https://www.youtube.com/feeds/videos.xml?channel_id=UCJps2S5PiabUY3yZv3iq0tw")
        ("https://www.youtube.com/feeds/videos.xml?channel_id=UCbJ1WFUdC4ImBlFReGNHjKQ")
        ("https://www.youtube.com/feeds/videos.xml?channel_id=UCvETBL47UPZVMBdIW-gFpPQ")
        ("https://www.youtube.com/feeds/videos.xml?channel_id=UCcGoqh8kLlACkFFpqXm6eSw")
        ("https://www.youtube.com/feeds/videos.xml?channel_id=UC224ep4hRGF54CFcwqapb4A")
        ("https://twitchrss.appspot.com/vod/dragonfriends")
        ("https://www.youtube.com/feeds/videos.xml?channel_id=UC8tThli1ZY7LW5Dxqr3Y0jA")
        ("https://sewmuchblack.de/feed/")
        ("https://updates.orgmode.org/feed/updates")
        ("https://blog.aaronbieber.com/posts/index.xml")
        ("https://www.tchwr.com/feed/")
        ("https://www.arp242.net/feed.xml")
        ("https://shellzine.net/feed/")
        ("https://notmyhostna.me/atom.xml")
        ("https://www.g-central.com/feed/")
        ("https://weather.gc.ca/rss/warning/nl-24_e.xml")
        ("https://www.youtube.com/feeds/videos.xml?channel_id=UCY3Nryf55m0yn48jLezBhlw")
        ("https://blog.jethro.dev/index.xml")
        ("https://www.kijiji.ca/rss-srp-buy-sell/st-johns/g-shock/k0c10l1700113")
        ("https://www.kijiji.ca/rss-srp-clothing-men/st-johns/small/c278l1700113a15183001?ad=offering")
        ("https://www.kijiji.ca/rss-srp-mens-shoes/st-johns/size+8__size+8+5/c15117001l1700113a15117001?ad=offering")
        ("https://www.kijiji.ca/rss-srp-buy-sell-desks/st-johns/desk/k0c239l1700113?ad=offering&for-sale-by=ownr")
        ("https://www.youtube.com/feeds/videos.xml?channel_id=UC1XDekTJ0jp24_aw4MncIsg")
        ("https://medium.com/feed/@ghostlux")
        ("https://idiotreport.substack.com/feed/")
        ("https://backstage.1blocker.com/feed")
        ("https://www.youtube.com/feeds/videos.xml?channel_id=UC8TjnmfivUw4bLB-VEn0_Sw")
        ("https://formerf1doc.wordpress.com/feed/")
        ("http://anaffordablewardrobe.blogspot.com/feeds/posts/default")
        ("http://feedpress.me/apt2024")
        ("https://sam217pa.github.io/index.xml")
        ("https://blog.blankbaby.com/atom.xml")
        ("https://cestlaz.github.io/rss.xml" emacs)
        ("http://blog.binchen.org/rss.xml" emacs)
        ("https://css-tricks.com/feed/")
        ("https://deathtrashgame.tumblr.com/rss")
        ("https://dieworkwear.com/rss")
        ("https://emacsredux.com/atom.xml" emacs)
        ("http://emacsrocks.com/atom.xml" emacs)
        ("https://fastmail.blog/rss/")
        ("https://epsalt.ca/rss" blog)
        ("https://hk-devblog.com/feed/")
        ("http://www.howardism.org/index.xml" emacs)
        ("http://feeds.feedburner.com/Ikeahacker")
        ("http://irreal.org/blog/?feed=rss2" emacs)
        ("https://www.kinowear.com/feed/")
        ("https://longreads.com/feed/")
        ("https://www.macsparky.com/blog?format=rss")
        ("http://mbork.pl/?action=rss" emacs)
        ("https://www.masteringemacs.org/feed" emacs)
        ("https://fuco1.github.io/rss.xml" emacs)
        ("https://oremacs.com/atom.xml")
        ("http://xenodium.com/rss.xml")
        ("https://mcmansionhell.com/rss")
        ("http://www.modernemacs.com/index.xml" emacs)
        ("https://nefariousreviews.com/feed/")
        ("https://genehack.blog/atom.xml")
        ("https://scifiinterfaces.com/feed/")
        ("https://updates.nonissue.org/rss")
        ("https://nullprogram.com/feed/" emacs)
        ("https://scripter.co/posts/index.xml" emacs)
        ("http://pragmaticemacs.com/feed/" emacs)
        ("http://www.lunaryorn.com/feed.atom" emacs)
        ("http://endlessparentheses.com/atom.xml" emacs)
        ("https://karl-voit.at/feeds/lazyblorg-all.atom_1.0.links-and-content.xml")
        ("https://sachachua.com/blog/feed/" emacs)
        ("https://feedpress.me/sixcolors")
        ("https://strattondelany.com/feed/" blog)
        ("https://www.stylesofman.com/feed/")
        ("http://takingnotenow.blogspot.com/feeds/posts/default")
        ("https://journal.styleforum.net/feed/")
        ("https://culturedcode.com/things/blog/feed/rss.xml")
        ("https://tungodies.com/feed/")
        ("https://manuel-uberti.github.io/feed" emacs)
        ("http://usuallywhatimdressed.in/feed/")
        ("https://zettelkasten.de/feed.atom")
        ("https://zzamboni.org/index.xml")
        ("https://eightiesandninetiesanime.tumblr.com/rss" image)
        ("https://1041uuu.tumblr.com/rss" image)
        ("https://www.drugsandwires.fail/feed/" comic)
        ("http://feeds.feedburner.com/Explosm" comic)
        ("https://www.foxtrot.com/feed/" comic)
        ("http://feeds.feedburner.com/PoorlyDrawnLines" comic)
        ("http://collet66.blog52.fc2.com/?xml")
        ("https://reddit-top-rss.herokuapp.com/?subreddit=deusex&averagePostsPerDay=2&view=rss")
        ("https://reddit-top-rss.herokuapp.com/?subreddit=cyberpunk&averagePostsPerDay=2&view=rss")
        ("https://reddit-top-rss.herokuapp.com/?subreddit=emacs&averagePostsPerDay=2&view=rss" emacs)
        ("https://reddit-top-rss.herokuapp.com/?subreddit=orgmode&averagePostsPerDay=2&view=rss" emacs)
        ("https://reddit-top-rss.herokuapp.com/?subreddit=techwearclothing&averagePostsPerDay=2&view=rss")
        ("https://reddit-top-rss.herokuapp.com/?subreddit=techwear&averagePostsPerDay=2&view=rss")
        ("https://reddit-top-rss.herokuapp.com/?subreddit=formula1&averagePostsPerDay=1&view=rss")
        ("https://reddit-top-rss.herokuapp.com/?subreddit=malefashionadvice&averagePostsPerDay=1&view=rss")
        ("https://reddit-top-rss.herokuapp.com/?subreddit=ClothingTechnology&averagePostsPerDay=1&view=rss")
        ("https://noonker.github.io/index.xml" emacs)
        ("https://mac.into.sh/index.xml")))
```

Honestly, it feels weird to share my entire collection of feeds in public. Like I'm sharing something very personal. Anyway, that's it. That's my Elfeed.


## Additional resources {#additional-resources}

-   [Elfeed Rules! Noonker — thoughts, guides, etc](https://noonker.github.io/posts/2020-04-22-elfeed/)
-   [Posts tagged elfeed « null program](https://nullprogram.com/tags/elfeed/)
-   [elfeed | Pragmatic Emacs](http://pragmaticemacs.com/category/elfeed/)
-   [Using Emacs - 29 -elfeed part 1 | C'est la Z](https://cestlaz-nikola.github.io/posts/using-emacs-29%20elfeed/)
-   [Using Emacs - 30 - elfeed part 2 - Hydras | C'est la Z](https://cestlaz-nikola.github.io/posts/using-emacs-30-elfeed-2/)
-   [Using Emacs - 31 - elfeed part 3 - macros | C'est la Z](https://cestlaz-nikola.github.io/posts/using-emacs-31-elfeed-3/)
