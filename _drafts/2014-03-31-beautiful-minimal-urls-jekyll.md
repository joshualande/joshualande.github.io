---
layout: post
title: How to Get Short Minimial URLs in Jekyll with Github Pages
comments: true
permalink: minimal-urls-jekyll
---

In this post, I will describe how to make the URLs for your blog
posts (what Jekyll calls permalinks) short and minimial.  For
Example, I wanted the URL for this post to be
[joshualande.com/minimial-urls-jekyll](joshualande.com/minimial-urls-jekyll)
without any tags, dates, or other extra characters.  This url is
short, looks nice, and is easy to share. I also read [some
speculation](http://davidtuite.com/posts/how-to-manage-permalinks-in-jekyll)
that smaller URLs help with
[SEO](http://en.wikipedia.org/wiki/Search_engine_optimization).

The [Jekyll documentation](http://jekyllrb.com/docs/permalinks/),
discusses the ability to set permalink to `none` in `_config.yml`
file:

```yaml
permalink: none
```

But this created URLs like
[joshualande.com/beautiful-minimal-urls-jekyll.html](joshualande.com/beautiful-minimal-urls-jekyll.html)
and I didn't want my URLs ending in ".html".

Next, I tried setting the permalink value to "/:title" in `_config.yml`:

```yaml
permalink: "/:title"
```

This appeared to work, but broke the static pages I had setup (like
[joshualande.com/about](joshualande.com/about)).  I could move all
the static pages to just be blog posts, but that would have broken
[ my archive](http://joshualande.com/archive/) of blog posts which
does not include static pages.

Finally, I realized that I could get my desired result without messing
up static pages by explicitly setting the `permalink` key
in the YAML metadata of each post.

```yaml
permalink: minimal-urls-jekyll
```

It is unfortunately that you have to explicitly set this at the top
of each blog post. But this does work, giving me beautiful URLs
like
[joshualande.com/minimal-urls-jekyll](joshualande.com/minimal-urls-jekyll)!
Please comment below if you know of a better way to do this.

{% include twitter_plug.html %}
