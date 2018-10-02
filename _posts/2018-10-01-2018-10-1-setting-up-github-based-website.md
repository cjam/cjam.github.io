---
title: Setting up my portfolio / blog
date: 2018-10-01 00:00:00 Z
layout: article
comments: true
description: My first post and subsequently the steps I took to get this site setup.
---

Ok, so here I am.  A first post discussing the very site holding this page.  Awesome.  So I am attempting to build a personal portfolio / blog automatically built (Using [Jekyll](https://jekyllrb.com/)) and hosted by [Github Pages](https://pages.github.com/).

## Setup

I ended up forking from my initial repo from <https://github.com/barryclark/jekyll-now>

Started with the [Instructions](https://pages.github.com/),  seemed reasonable. 

Ended up [installing Jekyll Locally](https://jekyllrb.com/docs/installation/) to try to speed up my iterating.

Started up the jekyll server

```sh
jekyll serve --livereload
```

Alright, after installing a few `gems` it seems that things are working as expected.

## Tweaks

Updated the `_config.yml`:
  - Updated name, email, avatar 
  - added links to various social media accounts
  - Integrated **disqus** for comments
  - Added google analytics

## Theme

I can tell already that I'm going to want a different theme.  Jekylls theme is just awesome.  I will do a bit of investigation into whats involved in changing themes.  

*~Investigation~*

Ok. so there are a lot of themes, I think I'll just stick with this one for now.  Ha....

I couldn't help it, so I started mashing together a few themes that I liked.  pulling out the code highlighting from Jekylls site and some of the article layout from the [Chalk Theme](http://chalk.nielsenramon.com/posts/introducing-chalk) which I thought looked simple and nice.

Ok, I think I may leave it at this.  Comitting my stuff to github and hoping that it builds. 🤞
