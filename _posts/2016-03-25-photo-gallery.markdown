---
layout: post
title:  "Photo Gallery"
categories: palm gadgets navigation

gallery_title: "palm navigator"
gallery:
  - background_image_path: assets/palmnavigator/01.jpg
    main_heading:
    data-lightbox: palmNavigator
    data-title:
    img-class: palmNavigator
  - background_image_path: assets/palmnavigator/02.jpg
    main_heading:
    data-lightbox: palmNavigator
    data-title:
    img-class: palmNavigator
  - background_image_path: assets/palmnavigator/03.jpg
    main_heading:
    data-lightbox: palmNavigator
    data-title:
    img-class: palmNavigator
  - background_image_path: assets/palmnavigator/04.jpg
    main_heading:
    data-lightbox: palmNavigator
    data-title:
    img-class: palmNavigator

    
---

{% if page.gallery != blank %}
<div class=".gallery">
<ul id="" class="list-unstyled row">
  {% for item in page.gallery %}
<li class="col-xs-6 col-sm-4 col-md-3">
<a href="{{ site.url }}/{{ item.background_image_path }}" target="_blank"> 
<img class="{{ item.img-class }}" src="{{ site.url }}/{{ item.background_image_path }}" alt=""/>
</a>
</li>
  {% endfor %}
</ul>
</div>
{% endif %}

