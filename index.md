---
layout: page
title: "Latest Articles"
excerpt: "Latest Articles"
modified: 2014-08-08T19:44:38.564948-04:00
---


<ul class="post-list">
{% for post in site.categories.blog %} 
  <li>
    <article>
      <a href="{{ site.url }}{{ post.url }}">
        <span class="entry-date">
          <time datetime="{{ post.date | date_to_xmlschema }}">{{ post.date | date: "%B %d, %Y" }}</time>
        </span>

        <strong>{{ post.title }}</strong>
        {% if post.excerpt %} <span class="excerpt">{{ post.excerpt | remove: '\[ ... \]' | remove: '\( ... \)' | markdownify | strip_html | strip_newlines | escape_once }}</span>{% endif %}
      </a>
    </article>
  </li>
{% endfor %}
</ul>
