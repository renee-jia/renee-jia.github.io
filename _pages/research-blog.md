---
title: "Research Blog"
permalink: /research-blog/
layout: single
author_profile: false
classes:
  - listing
---

# Research Blog

Longer pieces where I try to work something out properly — a problem I think is underspecified, the formalism it needs, and the part I still can't answer.

{% assign research_posts = site.posts | where_exp: "post", "post.categories contains 'Research Blog'" %}
{% for post in research_posts %}
{% include post-preview.html post=post %}
{% endfor %}

<p class="listing-note">More to come.</p>
