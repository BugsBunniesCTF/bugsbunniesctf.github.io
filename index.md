---
title: BugsBunnies
---
<h1>{{ page.title }}</h1>

<h2>Writeups</h2>

<ul>
    {% for post in site.posts %}
    <li>
        <h4><a href="{{ post.url }}">{{ post.ctf }} - {{ post.title }}</a></h4>
    </li>
    {% endfor %}
</ul>

<h2>Writeups 2</h2>
<ul>
  {% for ctf in site.ctfs %}
    {% for post in site.posts %}
      <h3>{{ ctf.name }}</h3>
      {% if post.ctf == ctf.short_name %}
        yes: {{ post.ctf }} == {{ ctf.short_name }}
      {% else %}
        nope: {{ post.ctf }} != {{ ctf.short_name }}
      {% endif %}
        <li>
          <h4><a href="{{ post.url }}">{{ post.ctf }} - {{ post.title }}</a></h4>
          {{ post.excerpt }}
        </li>
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

