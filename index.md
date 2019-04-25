---
layout: default
title: Blog
---
<h2>Blog
  <a class="fadedlink rss" href="http://feeds.feedburner.com/{{ site.feedburner_id }}" title="RSS feed">RSS Feed
  </a>
</h2>

<ul class="posts">
  {% for post in site.posts %}
    {% include post.html %}
  {% endfor %}
</ul>
