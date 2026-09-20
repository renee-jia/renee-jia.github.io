---
title: "Research Blog"
permalink: /research-blog/
layout: single
author_profile: false
classes:
  - listing
---

# Research Blog

Writing to figure out what I think.

{% assign research_posts = site.posts | where_exp: "post", "post.categories contains 'Research Blog'" %}

## Open problems

{% assign problem_posts = research_posts | where: "research_kind", "open-problem" %}
{% for post in problem_posts %}
{% include post-preview.html post=post %}
{% endfor %}

## How I think about AI

{% assign perspective_posts = research_posts | where: "research_kind", "perspective" %}
{% for post in perspective_posts %}
{% include post-preview.html post=post %}
{% endfor %}

{% capture unfiled %}{% for post in research_posts %}{% unless post.research_kind %}{% include post-preview.html post=post %}{% endunless %}{% endfor %}{% endcapture %}
{% assign unfiled_trimmed = unfiled | strip %}
{% if unfiled_trimmed.size > 0 %}
## Elsewhere

{{ unfiled_trimmed }}
{% endif %}

<p class="listing-note">More to come.</p>
