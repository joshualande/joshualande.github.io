---
layout: post
title: How to Get Minimial URLs in Jekyll with Github Pages
comments: true
permalink: minimal-urls-jekyll
---

Since I don't post that often and also don't intend to use my domain
[joshualande.com](joshualande.com) for other projects, I decided I
wanted to make the URLs (or what Jekyll calls permalinks) for my
blog URLs super minimial.  For example, for this post, I wanted the
URL to be [joshualande.com/minimial-urls-jekyll](joshualande.com/minimial-urls-jekyll).

This url is short, looks good, and is easy to share.  And I read
[some
speculation](http://davidtuite.com/posts/how-to-manage-permalinks-in-jekyll)
that smaller URLs help with
[SEO](http://en.wikipedia.org/wiki/Search_engine_optimization).

As is docuemnted [here](http://jekyllrb.com/docs/permalinks/), I
though I could do this by just setting the permalink to `none` in `_config.yml`
```yml
permalink: none
```

But this created URLs like
[joshualande.com/beautiful-minimal-urls-jekyll.html](joshualande.com/beautiful-minimal-urls-jekyll.html)
and I didn't want them ending in ".html".

Next, I tried setting the permalink value to "/:title", but this
broke the static pages I had setup (like
[joshualande.com/about](joshualande.com/about)).  I could move all
the static pages to just be blog posts, but that would break my
archive of blog posts
([http://joshualande.com/archive/](http://joshualande.com/archive/)).

Finally, I relized that I could get my desired result without messing
up static pages by explicitly setting the permalink in each blog
post.  For example, for this post I set at the top of the markdown
file:

> permalink: minimal-urls-jekyll

It is unfortunately that you have to explicilty set this at the top
of each blog post. But this does work, giving me beautiful URLs
like
[joshualande.com/minimal-urls-jekyll](joshualande.com/minimal-urls-jekyll).
Please comment below if you know a better way to do this.
