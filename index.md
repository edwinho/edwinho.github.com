---
layout: page
title: '振籽'
---
{% include JB/setup %}


<ul class="posts">
  {% for post in site.posts %}
    {% capture summary %}{{post.content | split:'<!--more-->' |first }}{% endcapture%}
    <div class="span12 row">
        <h2><a class="title" href="{{ BASE_PATH }}{{ post.url }}">{{ post.title }}</a></h2>
        <div class="post_at_index">
            {{summary}} 
        {% if summary != post.content %}<a href="{{ BASE_PATH }}{{ post.url }}" rel="nofollow">Read more...</a>{% endif %}
        </div>
        <br/>
        <div>
          <cite>{{ post.date | date: "%Y-%m-%d" }}</cite> <i class="icon-tag"></i>  {% for tag in post.tags %}<a href="{{ BASE_PATH }}{{ site.JB.tags_path }}#{{ tag }}-ref">{{ tag }}</a>{% if forloop.last %}{% else %}, {% endif %}{% endfor %}
       </div> 
        <div style="clear: both;"></div>
        <hr/>
    </div>
  {% endfor %}
</ul>


