---
title: "Paper Readings"
permalink: /paper-readings/
layout: single
author_profile: false
classes:
  - listing
---

# Paper Readings

Notes from papers I actually sat with — architecture, training tricks, and the occasional thing that changed how I read the next one.

## Transformer architecture

{% assign attn_res_post = site.posts | where_exp: "post", "post.title contains 'Attention Residuals'" | first %}
{% include post-preview.html post=attn_res_post %}

<p class="listing-note">More notes as I finish them.</p>
