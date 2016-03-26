---
layout: post
title:  "Palm Navigator"
date:   2016-03-25 12:09:56
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

This morning I noticed interesting lot on ebay, at first it seemed
to be a common Palm Pilot Professional by 3COM in original box with
manuals, software CDs and also with a modem included. Although
*modem* had same type of housing as Palm Modem, it had different
markings on it e.g. "Palm Navigator", "Precision Navigation,
Inc". So I became eager to find out more information what it is all
about. It was not a modem after all, but a sensor module, basically
what it turns your palm into electronic compass. This device has been
reviewed<sup>[[1]](http://the-gadgeteer.com/1998/05/03/palm_navigator_review/){:target="_blank"}</sup>
in the past and it has received good
feedback<sup>[[2]](http://www.verycomputer.com/21_5d90b8c63ca38561_1.htm){:target="_blank"}</sup>
I am wondering if software for this device is still available, since it is
possible to check it in demo mode even without having the compass itself.

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



references:  
[1][http://the-gadgeteer.com/1998/05/03/palm_navigator_review/](http://the-gadgeteer.com/1998/05/03/palm_navigator_review/){:target="_blank"}
[2][http://www.verycomputer.com/21_5d90b8c63ca38561_1.htm](http://www.verycomputer.com/21_5d90b8c63ca38561_1.htm){:target="_blank"}
