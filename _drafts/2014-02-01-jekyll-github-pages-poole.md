---
layout: page
title: Moving from Wordpress to Jekyll + Github Pages + poole
comments: true 
---

I recently migrated this website from [Wordpress](http://wordpress.com) to [Github pages](http://pages.github.com/), and I am really happy with the transition. Over time, I grew to really dislike how heavy-weight Wordpress is. I've never liked how when you write HTML in a WYSIWYG editor you aren't sure what code it is creating. The process of backing up my blog was a huge pain, as was having to constantly upgrade Wordpress to avoid scurity risks. And whenever I went in to edit anything, I was always afraid that I would fat-finger change somthing without realizing it. I hated having to mantain a opaque database in order to keep all of my text. And I was never sure how to dive into the HTML if I wanted to customize anything.

So when I learned about [Jekyll](http://jekyllrb.com/), it seemed like a great alternative. I like the idea that my entire blog is a set of static files. Besides simplicity, it makes backups so much easier and avoids most common security risks to a website. I could write my posts in [Markdown](http://en.wikipedia.org/wiki/Markdown) which I am used to from [Confluence](https://www.atlassian.com/software/confluence) and [GitHub](http://github.com). And  I was really excited that all of my blog posts would be static markdown files and I wouldn't have to deal with configuring a website. Also, markdown has the ability to really nicely embed coding examples. And finally, becasuse Jekyll is so light weight, I was able to give my blog the super-minimal and super-lightweight look I wanted.

The fact that Github provides [free hosting for Jekyll blogs](http://pages.github.com) is just icing on the cake. Besides having super easy version control of the blog, I can use Github's online editor to write blog posts and make minor modificaitons of the blog.

Instead of reading a lot of documentation, I found this really great git repo called [poole](https://github.com/poole/poole). poole provides "a clear and concise foundational setup for any Jekyll site. And it has a super minimal look which I was interested in. It does so by furnishing a full vanilla Jekyll install with example templates, pages, posts, and styles". So to get started with your own blog, all you have to do is great a new Git repo with the name <USERNAME>.github.io, download the poole repository, and push it into your repo. A few minutes later the website <USERNAME>.github.io will be ready. I only had a few posts on my previous website so I jsut copied them over manually, but there is [a package](http://jekyllrb.com/docs/migrations) for migrating blogs to Jekyll.

# Blog Layout

The initial state of the poole repository is:

{% highlight sh %}
$ ls -1
404.html
CNAME
LICENSE.md
README.md
_config.yml
_includes
_layouts
_posts
about.md
atom.xml
index.html
public
{% endhighlight %}

When you run Jekyll, it creates a folder called _site with the
static website. Every file or folder will get directly copied 
into the website except files or folders which begin with an underscore.
Markdown files will get automatically converted to HTML.

_posts contains all of the blog posts in markdown format:
{% highlight html %}
$ ls -1 _posts/
2013-12-31-whats-jekyll.md
2014-01-01-example-content.md
2014-01-02-introducing-poole.md
{% endhighlight %}
index.html contains the front page of the blog and about.md is a
plot post in markdown format (which can be viewed at DOMAIN.com/about).
If you want to have more static files, you can just add them to the
website.

_config contains general configuration stuff for the website.
{% highlight yaml %}
# Setup
title:            Poole
tagline:          'The Jekyll Butler'
description:      'Base theme for Jekyll themes by @mdo.'
url:              http://getpoole.com
...
{% endhighlight %}

Finally, the _layouts and _includes folders contain boiler-plate HTML for building the website.
It is not very hard to go in and modify those files to change the look and feel of the site.
{% highlight yaml %}
$ ls -1 _layouts/
default.html
page.html
post.html
$ ls -1 _includes/
head.html
{% endhighlight %}

# Customizations 

To get my blog to its current form, I made a few modifications to the base poole layout. 

FIrst, I wanted to create an [Archive](/archive) page which listed all of my blog posts.
To do this, I created the file [archive.md](https://github.com/joshualande/joshualande.github.io/blob/5e5ca6389fbc66be06488b9b7803e0278ee1b89f/archive.md) which shows a dynamic list of all of 
my blog posts:
{% highlight html %}
{% raw %}
---
layout: page
title: Archive
---

# Blog Posts

{% for post in site.posts %}
  * {{ post.date | date_to_string }} &raquo; [ {{ post.title }} ]({{ post.url }})
{% endfor %}
{% endraw %}
{% endhighlight %}

Next, I wanted to add a navigation bar at the top of the website with links to the [About](/about) and [Archive](/archive) page. To do this, I modified the file [_config.yml](https://github.com/joshualande/joshualande.github.io/blob/5e5ca6389fbc66be06488b9b7803e0278ee1b89f/_config.yml) to define a list of pages to show in my header:
{% highlight yaml %}
# This is the list of pages to incldue in the header of the website.
pages_list:       ['About', 'Archive']
{% endhighlight %}
I then modify the file [_layouts/default.html](https://github.com/joshualande/joshualande.github.io/blob/5e5ca6389fbc66be06488b9b7803e0278ee1b89f/_layouts/default.html) to loop over this list and create links to each of the pages on the website:
{% highlight html %}
<h3 class="masthead-title">
  <a href="/" title="Home">{{ site.title }}</a>
  {% for page_name in site.pages_list %}
    &nbsp;&nbsp;&nbsp;<small><a href="/{{ page_name | downcase }}">{{ page_name }}</a></small>
  {% endfor %}
</h3>
{% endhighlight %}



I wanted to enable [Disqus](http://disqus.com/) comments on the
blog. To do that, I modified the file _layouts/default.html to
include the line
{% highlight python %}
{% raw %}
{% include comments.html %}
{% endraw %}
{% endhighlight %}

And then I created a file _includes/comments.html which includes the HTML code given to my by Disqus:

{% highlight html %}
{% if page.comments %}
<!-- Add Disqus comments. -->
<div id="disqus_thread"></div>
<script type="text/javascript">
  /* * * CONFIGURATION VARIABLES: EDIT BEFORE PASTING INTO YOUR WEBPAGE * * */
  var disqus_shortname = '<USERNAME>'; // required: replace example with your forum shortname

  /* * * DON'T EDIT BELOW THIS LINE * * */
  (function() {
    var dsq = document.createElement('script'); dsq.type = 'text/javascript'; dsq.async = true;
    dsq.src = '//' + disqus_shortname + '.disqus.com/embed.js';
    (document.getElementsByTagName('head')[0] || document.getElementsByTagName('body')[0]).appendChild(dsq);
  })();
</script>
<noscript>Please enable JavaScript to view the <a href="http://disqus.com/?ref_noscript">comments powered by Disqus.</a></noscript>
<a href="http://disqus.com" class="dsq-brlink">comments powered by <span class="logo-disqus">Disqus</span></a>
{% endif %}
{% endhighlight %}
By setting up the code this way, I can enabled posts on a page-by-page basis to include comments by setting "comments: True" in the YAML header of a post.

And finally, I enabled Google Analtyics on the website... [TODO DOCUMENT]

If you have any questions about my implementation, you can view my website [on github](https://github.com/joshualande/joshualande.github.io)

Once I got the blog up to speed with the URL joshualande.github.io, it was easy to link [joshualande.com](http://joshualande.com) to it by following the instructions [here](http://davidensinger.com/2013/03/setting-the-dns-for-github-pages-on-namecheap).

# Links:

Here are some links which helped me along the way:
* The official [Jekyll](http://jekyllrb.com) website, along with detailed [documentation](http://jekyllrb.com/docs/home).
* Documentation by GitHub about hosting static webpages with [GitHub pages](http://pages.github.com).
* The [poole](https://github.com/poole/poole) GitHub repository.
...
