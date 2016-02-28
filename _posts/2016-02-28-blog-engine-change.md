---
layout: post
title: "Blog engine change"
date: 2016-02-28 20:30
---

Finally migrated blog from unmainained [nanoblogger](http://nanoblogger.sourceforge.net/) to
[jekyll](https://jekyllrb.com) + [github pages](https://pages.github.com/) +
[custom domain](https://help.github.com/articles/using-a-custom-domain-with-github-pages/).

RSS feeds and popular content redirected to new location with apache `mod_rewrite`:

~~~
RewriteEngine On
...
# blog migration
# redirection exceptions first
RewriteRule ^/favicon.(.*)$ - [L]
...
# feed redirects for feed readers
RewriteRule ^/blog/feed.xml$ http://dev.lab427.net/feed.xml [R=301,L]
RewriteRule ^/b/atom.xml$ http://dev.lab427.net/feed.xml [R=301,L]
RewriteRule ^/b/rss.xml$ http://dev.lab427.net/feed.xml [R=301,L]
# known/indexed articles
RewriteRule rpi_rtl_sdr_and_diy_discone http://dev.lab427.net/rpi-rtl_sdr-and-diy-discone.html [R=301,L]
...
# the rest
RewriteRule ^(.*)$ http://dev.lab427.net [R=301,L]
~~~
