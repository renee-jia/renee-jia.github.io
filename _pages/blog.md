---
title: "Blog"
permalink: /blog/
layout: single
author_profile: false
---

# Blog Posts

Welcome to my blog! Here I share my thoughts, research insights, and experiences in AI and machine learning.

## Generative AI and LLM

### Reward Hacking & Jailbreak Research
{% assign reward_hacking_post = site.posts | where_exp: "post", "post.title contains 'Reward Hacking'" | first %}
{% if reward_hacking_post %}
<div class="blog-post-preview" style="margin-bottom: 1.5em; padding: 1.2em; border: 1px solid #e9ecef; border-radius: 8px; background-color: #f8f9fa;">
  <h4 style="margin-top: 0; margin-bottom: 0.5em; font-size: 1.1em; font-weight: 600;">
    <a href="{{ reward_hacking_post.url }}" style="color: #1e3a8a; text-decoration: none;">{{ reward_hacking_post.title }}</a>
  </h4>
  <p style="margin: 0.5em 0; color: #666; font-size: 0.9em; text-align: left;">
    <i class="fas fa-calendar-alt" style="margin-right: 0.5em;"></i>
    {{ reward_hacking_post.date | date: "%B %d, %Y" }}
    {% if reward_hacking_post.read_time %}
    <span style="margin-left: 1em;">
      <i class="fas fa-clock" style="margin-right: 0.5em;"></i>
      {{ reward_hacking_post.read_time }}
    </span>
    {% endif %}
  </p>
  {% if reward_hacking_post.excerpt %}
  <p style="margin: 0.5em 0; line-height: 1.4; font-size: 0.9em; text-align: left;">
    {{ reward_hacking_post.excerpt | strip_html | truncatewords: 25 }}
  </p>
  {% endif %}
</div>
{% endif %}

### LLM Reasoning
{% assign llm_reasoning_post = site.posts | where_exp: "post", "post.title contains 'Reasoning in Large Language Models'" | first %}
{% if llm_reasoning_post %}
<div class="blog-post-preview" style="margin-bottom: 1.5em; padding: 1.2em; border: 1px solid #e9ecef; border-radius: 8px; background-color: #f8f9fa;">
  <h4 style="margin-top: 0; margin-bottom: 0.5em; font-size: 1.1em; font-weight: 600;">
    <a href="{{ llm_reasoning_post.url }}" style="color: #1e3a8a; text-decoration: none;">{{ llm_reasoning_post.title }}</a>
  </h4>
  <p style="margin: 0.5em 0; color: #666; font-size: 0.9em; text-align: left;">
    <i class="fas fa-calendar-alt" style="margin-right: 0.5em;"></i>
    {{ llm_reasoning_post.date | date: "%B %d, %Y" }}
    {% if llm_reasoning_post.read_time %}
    <span style="margin-left: 1em;">
      <i class="fas fa-clock" style="margin-right: 0.5em;"></i>
      {{ llm_reasoning_post.read_time }}
    </span>
    {% endif %}
  </p>
  {% if llm_reasoning_post.excerpt %}
  <p style="margin: 0.5em 0; line-height: 1.4; font-size: 0.9em; text-align: left;">
    {{ llm_reasoning_post.excerpt | strip_html | truncatewords: 25 }}
  </p>
  {% endif %}
</div>
{% endif %}

## Featured Series: Ranking & Recommendation Systems

### Long Sequence User Modeling in Ads Ranking
{% assign long_seq_post = site.posts | where_exp: "post", "post.title contains 'Long User Histories'" | first %}
{% if long_seq_post %}
<div class="blog-post-preview" style="margin-bottom: 1.5em; padding: 1.2em; border: 1px solid #e9ecef; border-radius: 8px; background-color: #f8f9fa;">
  <h4 style="margin-top: 0; margin-bottom: 0.5em; font-size: 1.1em; font-weight: 600;">
    <a href="{{ long_seq_post.url }}" style="color: #1e3a8a; text-decoration: none;">{{ long_seq_post.title }}</a>
  </h4>
  <p style="margin: 0.5em 0; color: #666; font-size: 0.9em; text-align: left;">
    <i class="fas fa-calendar-alt" style="margin-right: 0.5em;"></i>
    {{ long_seq_post.date | date: "%B %d, %Y" }}
    {% if long_seq_post.read_time %}
    <span style="margin-left: 1em;">
      <i class="fas fa-clock" style="margin-right: 0.5em;"></i>
      {{ long_seq_post.read_time }}
    </span>
    {% endif %}
  </p>
  {% if long_seq_post.excerpt %}
  <p style="margin: 0.5em 0; line-height: 1.4; font-size: 0.9em; text-align: left;">
    {{ long_seq_post.excerpt | strip_html | truncatewords: 25 }}
  </p>
  {% endif %}
</div>
{% endif %}

### Sequential Learning in Ranking AI
{% assign sequential_post = site.posts | where_exp: "post", "post.title contains 'Sequential Learning'" | first %}
{% if sequential_post %}
<div class="blog-post-preview" style="margin-bottom: 1.5em; padding: 1.2em; border: 1px solid #e9ecef; border-radius: 8px; background-color: #f8f9fa;">
  <h4 style="margin-top: 0; margin-bottom: 0.5em; font-size: 1.1em; font-weight: 600;">
    <a href="{{ sequential_post.url }}" style="color: #1e3a8a; text-decoration: none;">{{ sequential_post.title }}</a>
  </h4>
  <p style="margin: 0.5em 0; color: #666; font-size: 0.9em; text-align: left;">
    <i class="fas fa-calendar-alt" style="margin-right: 0.5em;"></i>
    {{ sequential_post.date | date: "%B %d, %Y" }}
    {% if sequential_post.read_time %}
    <span style="margin-left: 1em;">
      <i class="fas fa-clock" style="margin-right: 0.5em;"></i>
      {{ sequential_post.read_time }}
    </span>
    {% endif %}
  </p>
  {% if sequential_post.excerpt %}
  <p style="margin: 0.5em 0; line-height: 1.4; font-size: 0.9em; text-align: left;">
    {{ sequential_post.excerpt | strip_html | truncatewords: 25 }}
  </p>
  {% endif %}
</div>
{% endif %}

### Contemporary Recommendation Systems
{% assign contemporary_post = site.posts | where_exp: "post", "post.title contains 'Contemporary RecSys'" | first %}
{% if contemporary_post %}
<div class="blog-post-preview" style="margin-bottom: 1.5em; padding: 1.2em; border: 1px solid #e9ecef; border-radius: 8px; background-color: #f8f9fa;">
  <h4 style="margin-top: 0; margin-bottom: 0.5em; font-size: 1.1em; font-weight: 600;">
    <a href="{{ contemporary_post.url }}" style="color: #1e3a8a; text-decoration: none;">{{ contemporary_post.title }}</a>
  </h4>
  <p style="margin: 0.5em 0; color: #666; font-size: 0.9em; text-align: left;">
    <i class="fas fa-calendar-alt" style="margin-right: 0.5em;"></i>
    {{ contemporary_post.date | date: "%B %d, %Y" }}
    {% if contemporary_post.read_time %}
    <span style="margin-left: 1em;">
      <i class="fas fa-clock" style="margin-right: 0.5em;"></i>
      {{ contemporary_post.read_time }}
    </span>
    {% endif %}
  </p>
  {% if contemporary_post.excerpt %}
  <p style="margin: 0.5em 0; line-height: 1.4; font-size: 0.9em; text-align: left;">
    {{ contemporary_post.excerpt | strip_html | truncatewords: 25 }}
  </p>
  {% endif %}
</div>
{% endif %}

### Deep Learning Era in Recommendation Systems
{% assign deeplearning_post = site.posts | where_exp: "post", "post.title contains 'Deep Learning Era'" | first %}
{% if deeplearning_post %}
<div class="blog-post-preview" style="margin-bottom: 1.5em; padding: 1.2em; border: 1px solid #e9ecef; border-radius: 8px; background-color: #f8f9fa;">
  <h4 style="margin-top: 0; margin-bottom: 0.5em; font-size: 1.1em; font-weight: 600;">
    <a href="{{ deeplearning_post.url }}" style="color: #1e3a8a; text-decoration: none;">{{ deeplearning_post.title }}</a>
  </h4>
  <p style="margin: 0.5em 0; color: #666; font-size: 0.9em; text-align: left;">
    <i class="fas fa-calendar-alt" style="margin-right: 0.5em;"></i>
    {{ deeplearning_post.date | date: "%B %d, %Y" }}
    {% if deeplearning_post.read_time %}
    <span style="margin-left: 1em;">
      <i class="fas fa-clock" style="margin-right: 0.5em;"></i>
      {{ deeplearning_post.read_time }}
    </span>
    {% endif %}
  </p>
  {% if deeplearning_post.excerpt %}
  <p style="margin: 0.5em 0; line-height: 1.4; font-size: 0.9em; text-align: left;">
    {{ deeplearning_post.excerpt | strip_html | truncatewords: 25 }}
  </p>
  {% endif %}
</div>
{% endif %}

### Classic Foundations Recommendation Systems
{% assign foundational_post = site.posts | where_exp: "post", "post.title contains 'Foundational Papers'" | first %}
{% if foundational_post %}
<div class="blog-post-preview" style="margin-bottom: 1.5em; padding: 1.2em; border: 1px solid #e9ecef; border-radius: 8px; background-color: #f8f9fa;">
  <h4 style="margin-top: 0; margin-bottom: 0.5em; font-size: 1.1em; font-weight: 600;">
    <a href="{{ foundational_post.url }}" style="color: #1e3a8a; text-decoration: none;">{{ foundational_post.title }}</a>
  </h4>
  <p style="margin: 0.5em 0; color: #666; font-size: 0.9em; text-align: left;">
    <i class="fas fa-calendar-alt" style="margin-right: 0.5em;"></i>
    {{ foundational_post.date | date: "%B %d, %Y" }}
    {% if foundational_post.read_time %}
    <span style="margin-left: 1em;">
      <i class="fas fa-clock" style="margin-right: 0.5em;"></i>
      {{ foundational_post.read_time }}
    </span>
    {% endif %}
  </p>
  {% if foundational_post.excerpt %}
  <p style="margin: 0.5em 0; line-height: 1.4; font-size: 0.9em; text-align: left;">
    {{ foundational_post.excerpt | strip_html | truncatewords: 25 }}
  </p>
  {% endif %}
</div>
{% endif %}

## Recent Posts

{% assign posts = site.posts | sort: 'date' | reverse %}
{% for post in posts %}
  {% unless post.categories contains 'AI Learning Guide' %}
<div class="blog-post-preview" style="margin-bottom: 2em; padding: 1.5em; border: 1px solid #e9ecef; border-radius: 8px; background-color: #f8f9fa;">
  <h3 style="margin-top: 0; margin-bottom: 0.5em;">
    <a href="{{ post.url }}" style="color: #1e3a8a; text-decoration: none;">{{ post.title }}</a>
  </h3>
  <p style="margin: 0.5em 0; color: #666; font-size: 0.9em; text-align: left;">
    <i class="fas fa-calendar-alt" style="margin-right: 0.5em;"></i>
    {{ post.date | date: "%B %d, %Y" }}
    {% if post.read_time %}
    <span style="margin-left: 1em;">
      <i class="fas fa-clock" style="margin-right: 0.5em;"></i>
      {{ post.read_time }}
    </span>
    {% endif %}
    {% if post.categories %}
    <span style="margin-left: 1em;">
      <i class="fas fa-folder" style="margin-right: 0.5em;"></i>
      {{ post.categories | join: ", " }}
    </span>
    {% endif %}
  </p>
  {% if post.excerpt %}
  <p style="margin: 0.5em 0; line-height: 1.5; text-align: left;">
    {{ post.excerpt | strip_html | truncatewords: 30 }}
  </p>
  {% endif %}
  <a href="{{ post.url }}" style="color: #1e3a8a; text-decoration: none; font-size: 0.9em;">
    Read more <i class="fas fa-arrow-right" style="margin-left: 0.3em;"></i>
  </a>
</div>
  {% endunless %}
{% endfor %}

---

*More posts coming soon! Feel free to reach out if you have any questions or suggestions.*
