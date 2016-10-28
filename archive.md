---
layout: default
title: Blog's archive 
comments: False
---
{% include header.html type="page" %}

<div class="container" role="main">
  <div class="row">
    <div class="col-lg-8 col-lg-offset-2 col-md-10 col-md-offset-1">
      <h3>This year's posts</h3>
      {% for post in site.posts %}
          {% unless post.next %}
          <ul class="this">     
          {% else %}
               {% capture year %}{{ post.date | date: '%Y' }}{% endcapture %}
               {% capture nyear %}{{ post.next.date | date: '%Y' }}{% endcapture %}
               {% if year != nyear %}
                   </ul>
                   <h3>{{ post.date | date: '%Y' }}</h3>
                   <ul class="past">
               {% endif %}
          {% endunless %}
          <li><time>{{ post.date | date:"%d %b" }}</time> - <a href="{{ post.url }}">{{ post.title }}</a></li>
      {% endfor %}
      </ul>
      {% if page.comments %}
        <div class="disqus-comments">
          {% include disqus.html %}
        </div>
      {% endif %}
    </div>
  </div>
</div>

