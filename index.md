---
layout: default
title: Blog
---

Hey! I'm Juan de Bravo, a Director of Engineering. Welcome to my blog where I share insights about software development, team leadership, and technical challenges.

### Most Recent Posts

<ul class="posts">
  {% for post in site.posts limit:site.recent_posts_limit %}
    {% include post.html %}
  {% endfor %}
</ul>

<div style="display: flex; align-items: center;">
  <h3>All Posts</h3>
  <a class="fadedlink rss" href="http://feeds.feedburner.com/{{ site.feedburner_id }}" title="RSS feed" style="height: 35px; margin-left: 10px;">RSS Feed</a>
</div>

<ul class="posts">
  {% for post in site.posts offset:site.recent_posts_limit %}
    {% include post.html %}
  {% endfor %}
</ul>
