---
title: "Paper Readings"
permalink: /paper-readings/
layout: single
author_profile: false
classes:
  - listing
---

# Paper Readings

Reading closely rather than widely.

## Transformer architecture

{% assign attn_res_post = site.posts | where_exp: "post", "post.title contains 'Attention Residuals'" | first %}
{% include post-preview.html post=attn_res_post %}

<p class="listing-note">More to come.</p>
