---
layout: post
title: How to Redirect URLs in Jekyll Using Github Pages and the jekyll-redirect-from Plugin
comments: true
permalink: redirect-urls-jekyll-github
---

When changing the permalink structure of your blog or migrating
your blog from one blogging platform to another, you often want to
change the permalink structure of your blog but keep the old
URLs and have them redirect to the new pages.
This will allow you to preserve your 
Google search rankings, tweet history, SEO, etc.

Fortunately, Github Pages makes this really
easy because they
[recently](https://github.com/blog/1797-repository-metadata-and-plugin-support-for-github-pages)
enabled support for the
[jekyll-redirect-from](https://github.com/jekyll/jekyll-redirect-from)
plugin.  To get these URLs to redirect, all you have to do is:

* Install jekyll-redirect-from. To do that, I ran the command

  ```yaml
  gem install jekyll-redirect-from --source http://rubygems.org
  ```
  
* Include this plugin in Jekyll by adding to your `_config.yml`:

  ```yaml
  gems:
    - jekyll-redirect-from
  ```

* Add to the YAML metadata of the post a line describing the old URL which should
  be redirected.
  For example, if the
  original URL for this post was
  [joshualande.com/2014/04/02/minimal-urls-jekyll](joshualande.com/2014/04/02/minimal-urls-jekyll),
  then we could redirect that URL to the new URL 
  by adding a line to the YAML metadata of this blog post:
  
  ```yaml
  redirect_from: "/2014/04/02/minimal-urls-jekyll/"
  ```
  
  jekyll-redirect-from is further documented [here](https://help.github.com/articles/redirects-on-github-pages).

* Finally, if you use [DISQUS](http://disqus.com/) to host comments
  on your blog, you will have to explicitly tell DISQUS that you
  changed the URLs for your blog posts.  DISQUS provide an easy URL
  Mapper which is documented
  [here](http://help.disqus.com/customer/portal/articles/912757-url-mapper).

{% include twitter_plug.html %}
