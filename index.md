---
layout: page
title: '振籽'
---
{% include JB/setup %}


<ul class="posts">
  {% for post in site.posts %}
    <article class="post_item post">
    <header>
      <h1><a href="{{ post.url }}">{{ post.title }}</a></h1>
      <time datetime="{{ post.date|date:'%Y-%m-%d' }}">{{ post.date | date_to_long_string }}</time>
    </header>
    {{ post.content }}
    <footer><a href="{{ post.url }}#disqus_thread" class="comments_link">Comments</a></footer>
  </article>
  {% endfor %}
</ul>


