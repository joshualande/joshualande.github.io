---
layout: page
title: Moving from Wordpress to Jekyll + Github Pages
comments: true
---

I recently migrated this website from [Wordpress](http://wordpress.com) to [Github pages](http://pages.github.com/), and I am really happy with the trnasition. Over time, I grew to really dislike how heavy-weight Wordpress is. I've never liked how when you write HTML in a WYSIWYG editor you aren't sure what code it is creating. The process of backing up my blog was a huge pain, as was having to constantly upgrade Wordpress to avoid scurity risks. And whenever I went in to edit anything, I was always afraid that I would fat-finger change somthing without realizing it. I hated having to mantain a opaque database in order to keep all of my text. And I was never sure how to dive into the HTML if I wanted to customize anything.

So when I learned about [Jekyll](http://jekyllrb.com/), it seemed like a great alternative. I like the idea that my entire blog is a set of static files. Besides simplicity, it makes backups so much easier and avoids most common security risks to a website. I could write my posts in [Markdown](http://en.wikipedia.org/wiki/Markdown) which I am used to from [Confluence](https://www.atlassian.com/software/confluence) and [GitHub](http://github.com). And  I was really excited that all of my blog posts would be static markdown files and I wouldn't have to deal with configuring a website. And the fact that Github provided [free hosting for Jekyll blogs](http://pages.github.com) was just icing on the cake. And finally, becasuse Jekyll is so light weight, I was able to give my blog the super-minimal and super-lightweight look I wanted.


Backing up a Jekyll blog is trivial.

The process itself for getting set up is really easy

https://github.com/joshualande/joshualande.github.io

Once I got the URL joshualande.github.io working, it was easy to link [joshualande.com](http://joshualande.com) to it by following the instructions [here](http://davidensinger.com/2013/03/setting-the-dns-for-github-pages-on-namecheap).

Make a note about how I used GitHub Pages.

Base website:

* http://hyde.getpoole.com
* https://github.com/poole/poole#usage
...
