---
title: BugsBunnies
---
<h1>{{ page.title }}</h1>

<h2>Writeups</h2>
<ul>
  {% for ctf in site.ctfs %}
    {% for post in site.posts %}
      <h3>{{ ctf.name }}</h3>
      {% if post.ctf == ctf.short_name %}
        <li>
          <h4><a href="{{ post.url }}">{{ post.title }}</a></h4>
          <!-- {{ post.excerpt }} -->
        </li>
      {% endif %}
    {% endfor %}
  {% endfor %}
</ul>

<h2>Members</h2>
<ul>
  {% for author in site.authors %}
    <li>
      <h2>{{ author.name }}</h2>
      <p>{{ author.content | markdownify }}</p>
    </li>
  {% endfor %}
</ul>

