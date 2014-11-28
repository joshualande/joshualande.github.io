---
layout: post
title: How to Set Short URLs in Jekyll with Github Pages
comments: true
permalink: short-urls-jekyll
---

In this post, I will describe how to make the URLs for your blog
posts (what Jekyll calls permalinks) short and minimal.  For
example, I wanted the URL for this post to be
[{{ site.shorturl }}/short-urls-jekyll]({{ site.url }}/short-urls-jekyll)
without any tags, dates, or other extra characters.  This url is
short, looks nice, and is easy to share. I also read [some
speculation](http://davidtuite.com/posts/how-to-manage-permalinks-in-jekyll)
that smaller URLs help with
[SEO](http://en.wikipedia.org/wiki/Search_engine_optimization).

The [Jekyll documentation](http://jekyllrb.com/docs/permalinks/),
discusses the ability to set permalink to `none` in the `_config.yml`
file:

```yaml
permalink: none
```

But this created URLs like
[{{ site.shorturl }}/short-urls-jekyll.html]({{ site.url }}/short-urls-jekyll.html)
and I didn't want my URLs ending in ".html".

Next, I tried setting the permalink value to "/:title" in `_config.yml`:

```yaml
permalink: "/:title"
```

This appeared to work, but broke the static pages I had setup (like
[{{ site.shorturl }}/about]({{ site.url }}/about)).  I could move all
the static pages to just be blog posts, but that would have broken
[my archive]({{ site.url }}/archive/) of blog posts which
should not include static pages.

Finally, I realized that I could get my desired result without messing
up static pages by explicitly setting the `permalink` key
in the YAML metadata of each post. For this post, I set:

```yaml
permalink: short-urls-jekyll
```

It is unfortunate that you have to explicitly set this at the top
of each blog post. But this does work, giving me beautiful URLs
like
[{{ site.shorturl }}/short-urls-jekyll]({{ site.url }}/short-urls-jekyll).
Please comment below if you know of a simpler way to way to do this.

Finally, if you already have existing permalinks and want them
to redirect to your new url, you can do this by following the
instructions on [this blog post]({% post_url 2014-04-02-redirect-permalink-jekyll-github %}).

{% include twitter_plug.html %}
