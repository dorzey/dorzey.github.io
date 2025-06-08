---
layout: page
title: My Talks
permalink: /talks/
---

<div class="talks">
  {% assign sorted_talks = site.talks | sort: 'date' | reverse %}
  {% for talk in sorted_talks %}
    <article class="talk">
      <h2>
        <a href="{{ talk.url | relative_url }}">{{ talk.title }}</a>
      </h2>
      <p class="talk-meta">{{ talk.date | date: "%B %-d, %Y" }}</p>
      {{ talk.excerpt }}
    </article>
  {% endfor %}
</div> 