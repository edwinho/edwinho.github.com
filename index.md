---
layout: page
title: '振籽'
---
{% include JB/setup %}


<ul class="posts">
  {% for post in site.posts %}
    <h1><a class="title" href="{{ BASE_PATH }}{{ post.url }}">{{ post.title }}</a></h1>
        <br/>
        <div>
          <cite>{{ post.date | date: "%Y-%m-%d" }}</cite> <i class="icon-tag"></i>  {% for tag in post.tags %}<a href="{{ BASE_PATH }}{{ site.JB.tags_path }}#{{ tag }}-ref">{{ tag }}</a>{% if forloop.last %}{% else %}, {% endif %}{% endfor %}
  {% endfor %}
</ul>


