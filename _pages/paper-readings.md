---
title: "Paper Readings"
permalink: /paper-readings/
layout: single
author_profile: false
---

# Paper Readings

A collection of my notes and summaries from reading research papers in AI, machine learning, and related fields.

## Transformer Architecture

### Attention Residuals
{% assign attn_res_post = site.posts | where_exp: "post", "post.title contains 'Attention Residuals'" | first %}
{% if attn_res_post %}
<div class="blog-post-preview" style="margin-bottom: 1.5em; padding: 1.2em; border: 1px solid #e9ecef; border-radius: 8px; background-color: #f8f9fa;">
  <h4 style="margin-top: 0; margin-bottom: 0.5em; font-size: 1.1em; font-weight: 600;">
    <a href="{{ attn_res_post.url }}" style="color: #1e3a8a; text-decoration: none;">{{ attn_res_post.title }}</a>
  </h4>
  <p style="margin: 0.5em 0; color: #666; font-size: 0.9em; text-align: left;">
    <i class="fas fa-calendar-alt" style="margin-right: 0.5em;"></i>
    {{ attn_res_post.date | date: "%B %d, %Y" }}
    {% if attn_res_post.read_time %}
    <span style="margin-left: 1em;">
      <i class="fas fa-clock" style="margin-right: 0.5em;"></i>
      {{ attn_res_post.read_time }}
    </span>
    {% endif %}
  </p>
  {% if attn_res_post.excerpt %}
  <p style="margin: 0.5em 0; line-height: 1.4; font-size: 0.9em; text-align: left;">
    {{ attn_res_post.excerpt | strip_html | truncatewords: 25 }}
  </p>
  {% endif %}
</div>
{% endif %}

---

*More paper readings coming soon!*
