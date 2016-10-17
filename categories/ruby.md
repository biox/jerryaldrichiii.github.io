---
layout: default
title: Ruby
---

<div id="main">
  {% assign posts = site.posts | where:'category', 'ruby'%}
  {% for post in posts %}
    <section>
      <header>
	<h3><a href="{{ post.url }}">{{ post.title }}</a></h3>
      </header>
      <content>
	<p><strong>{{ post.summary }}</strong></p>
      </content>
    </section>
  {% endfor %}
</div>
