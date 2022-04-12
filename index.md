---
title: BugsBunnies
---
<h1>{{ page.title }}</h1>

<h2>Writeups</h2>
<ul>
  
  {% for ctf in site.ctf %}
    {% for post in site.posts %}
      <h3>{{ ctf.name }}</h3>
      {% if post.ctf == ctf.short_name %}
        <li>
          <h4><a href="{{ post.url }}">{{ post.ctf }} - {{ post.title }}</a></h2>
          {{ post.excerpt }}
        </li>
      {% endif %}
    {% endfor %}
  {% endfor %}
</ul>

<h2>Member</h2>
<ul>
  {% for author in site.authors %}
    <li>
      <h2>{{ author.name }}</h2>
      <p>{{ author.content | markdownify }}</p>
    </li>
  {% endfor %}
</ul>

