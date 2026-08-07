---
layout: default
title: News
---

# League News

{% if site.posts.size == 0 %}
<p>No news yet — add a file to <code>_posts/</code> to publish your first update.</p>
{% endif %}

<ul class="post-list">
{% for post in site.posts %}
  <li>
    <span class="post-date">{{ post.date | date: "%b %-d, %Y" }}</span>
    <a href="{{ post.url | relative_url }}">{{ post.title }}</a>
    {% if post.excerpt %}<p>{{ post.excerpt | strip_html | truncate: 160 }}</p>{% endif %}
  </li>
{% endfor %}
</ul>
