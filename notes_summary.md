---
layout: default
---
<!--
  In notes, only put header on those
  that you want to show up in categories
  and tags
-->
<div class="posts">
  {% for post in site.categories.notes %}
  <div class="post">
    <h1 class="post-title">
      <a href="{{ post.url | absolute_url }}">
        {{ post.title }}
      </a>
    </h1>

    <span class="post-date">{{ post.date | date_to_string }}</span>

    {{ post.excerpt }}
    <BR/>
    <a href="{{ site.baseurl }}{{ post.url }}" class="read-more">Read More</a>
  </div>
  {% endfor %}
</div>