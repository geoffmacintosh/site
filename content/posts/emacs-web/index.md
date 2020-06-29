+++
title = "Emacs Web"
author = ["Geoff MacIntosh"]
date = 2020-06-29T00:00:00-02:30
tags = ["emacs"]
draft = false
+++

I don't use Emacs to browse the web, although it can be done, and I guess people do. I do a few browser-related things in it though, including storing my bookmarks in an Org-mode file and such. Here's what I've changed.

Customizing the browse-url handlers is remarkably powerful. I don't use Emacs as a web browser much, but I do use a lot of links in Org-mode documents. If something isn't set here, it opens the URL in the default manner, which in my case is Safari ([Technology Preview](https://developer.apple.com/safari/technology-preview/)).

```emacs-lisp
(use-package browse-url
  :straight nil
  :custom
  (browse-url-handlers '(("wikipedia"   . eww )
                         ("youtu\\.?be" . actuator-browse-video)
                         ("twitch"      . actuator-browse-video))))
```

I want video links to be opened in MPV. This helps my battery life as well as my personal life because I don't have to visit YouTube. This requires [MPV](https://mpv.io) to be installed, which is best installed via [Brew](http://brew.sh) on macOS. I've tried to use [Nix](https://nixos.org/download.html), but it doesn't work well.

```emacs-lisp
  (defun actuator-browse-video (url &rest _args)
    "Browse a URL with a dedicated video player.
Avoids opening a browser window."
    (start-process "mpv" nil "mpv" url))
```

SHR is used to render all sorts of basic HTML in Emacs, including [Elfeed]({{< relref "elfeed" >}}) posts and Nov.el books. Normally it wraps at the page width, but that can be adjusted.

```emacs-lisp
(use-package shr
  :straight nil
  :custom
  (shr-width 75))
```
