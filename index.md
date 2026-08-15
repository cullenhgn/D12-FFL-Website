---
layout: default
title: News
---

# News

{% if site.posts.size == 0 %}
<p>No news yet — add a file to <code>_posts/</code> to publish your first update.</p>
{% endif %}

<ul class="post-list">
{% for post in site.posts %}
  <li>
    <span class="post-date">{{ post.date | date: "%b %-d, %Y" }}</span>
    <h2 class="post-list-title">{{ post.title }}</h2>
    <div class="post-list-content">{{ post.content }}</div>
  </li>
{% endfor %}
</ul>
