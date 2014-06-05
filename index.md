---
layout: default
title: Livlong En Prospa
tagline: Random Words
---
<div class="intro">
From the times of late last year(, or the one before)*n have I been searching for a good way to flat map the sequence of options life was sent with from an unknown sender, called The Sender. The Sender is still awaiting response. It will have to hold on for quite a while longer.
</div>
### Posts

<ul class="posts">
  {% for post in site.posts %}
    <li>
      <a class="posttitle" href="{{ post.url }}">{{ post.title }}</a>
      <div class="excerpt">{{ post.excerpt }}</div>
    </li>
  {% endfor %}
</ul>
