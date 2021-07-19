---
layout: post
title: "Optimizing the browser experience"
author: Bharathi Ramana Joshi
categories: tips
tags: [live-list]
image:
---

This post contains a list of tricks I employ to make my browser usage more
efficient. All of these should work on Chromium (and thus Chrome) as it is my
primary browser and I use these there (I was a Firefox user until it started
using noticeably more resources, if anybody knows how to improve Firefox's
performance please [let me know](/menu/about.html)), however most of them will
work on Firefox (and most likely, other browsers) as well. They take barely ~15
minutes to set up but have saved me a **lot** of time, so it is definitely worth
investing those 15 minutes.

## Browser keyboard shortcuts

Here are my top 5 most used ones

- `Ctrl+l`: switching to url bar from content
- `Ctrl+t`: open new tab
- `Ctrl+w`: close current tab
- `Ctrt+Shift+t`: reopen last closed tab
- `Ctrl/Alt+<number>`: switch to tab

A full list can be found
[here](https://support.google.com/chrome/answer/157179?hl=en)

## Search engines

If you go to [chrome://settings/searchEngines](chrome://settings/searchEngines)
in Chromium ([about:preferences#search](about:preferences#search) in Firefox),
you can add shortcuts for search engines. For instance, I have `w` set to the
following at the moment

```
https://en.wikipedia.org/w/index.php?title=Special:Search&search=%s
```

Now if I wanted to go to the Wikipedia page on Red Hot Chilli Peppers, I'd have
to enter

```
w Red Hot Chilli Peppers
```

in the url bar instead of going to Wikipedia, clicking the search bar and
entering my query.

I've also found having a `leaveAddressBar` engine to be very useful. This is to
switch back to the webpage from the url bar, without having to click or press
tab multiple times (taken from [here](https://superuser.com/a/324267)).

## Plugins

### Vimium

This plugin lets me use Vim bindings inside Chromium, making it convenient to
employ my vim muscle memory. It has keyboard bindings for almost everything -
scrolling, navigating tabs, opening links, inserting text in forms, searching
etc. The [GitHub README](https://github.com/philc/vimium#keyboard-bindings)
lists all features.

### Adblock Plus

This blocks most ads

### uBlock Origin

This allows me to enforce my own content filtering choices - I can hide things I
don't want to see etc
