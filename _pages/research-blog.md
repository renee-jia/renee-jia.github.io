---
title: "Research Blog"
permalink: /research-blog/
layout: single
author_profile: false
classes:
  - listing
---

# Research Blog

Where I try to work something out properly — a problem I think is underspecified, the formalism it needs, and the part I still can't answer. Some of these I got to the end of. Some I'm still circling.

{% assign research_posts = site.posts | where_exp: "post", "post.categories contains 'Research Blog'" %}

## Worked through

{% assign worked_posts = research_posts | where: "research_stage", "worked-through" %}
{% for post in worked_posts %}
{% include post-preview.html post=post %}
{% endfor %}

## Still open

{% assign open_posts = research_posts | where: "research_stage", "still-open" %}
{% for post in open_posts %}
{% include post-preview.html post=post %}
{% endfor %}

{% capture unfiled %}{% for post in research_posts %}{% unless post.research_stage %}{% include post-preview.html post=post %}{% endunless %}{% endfor %}{% endcapture %}
{% assign unfiled_trimmed = unfiled | strip %}
{% if unfiled_trimmed.size > 0 %}
## Elsewhere

{{ unfiled_trimmed }}
{% endif %}

<p class="listing-note">More to come.</p>
