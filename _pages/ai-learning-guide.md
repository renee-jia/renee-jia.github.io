---
title: "AI Learning Guide"
permalink: /ai-learning-guide/
layout: single
author_profile: false
classes:
  - listing
---

# AI Learning Guide

This is less a curriculum than a pile of notes I keep nearby — the pieces I wish I'd had in one place when I was first getting a footing, and a few later ones I still go back to.

## Start here

{% assign beginner_post = site.posts | where_exp: "post", "post.title contains 'AI Beginner'" | first %}
{% include post-preview.html post=beginner_post %}

## Ranking and recommendation

{% assign foundational_post = site.posts | where_exp: "post", "post.title contains 'Foundational Papers'" | first %}
{% include post-preview.html post=foundational_post %}

{% assign deeplearning_post = site.posts | where_exp: "post", "post.title contains 'Deep Learning Era'" | first %}
{% include post-preview.html post=deeplearning_post %}

{% assign sequential_post = site.posts | where_exp: "post", "post.title contains 'Sequential Learning'" | first %}
{% include post-preview.html post=sequential_post %}

{% assign contemporary_post = site.posts | where_exp: "post", "post.title contains 'Contemporary RecSys'" | first %}
{% include post-preview.html post=contemporary_post %}

{% assign long_seq_post = site.posts | where_exp: "post", "post.title contains 'Long User Histories'" | first %}
{% include post-preview.html post=long_seq_post %}

## Language models and reasoning

{% assign llm_reasoning_post = site.posts | where_exp: "post", "post.title contains 'Reasoning in Large Language Models'" | first %}
{% include post-preview.html post=llm_reasoning_post %}

## Training signals, reward, and alignment

{% assign reward_hacking_post = site.posts | where_exp: "post", "post.title contains 'Reward Hacking'" | first %}
{% include post-preview.html post=reward_hacking_post %}

{% assign credit_post = site.posts | where_exp: "post", "post.title contains 'Final-Outcome Rewards'" | first %}
{% include post-preview.html post=credit_post %}

{% assign placed = "AI Beginner,Foundational Papers,Deep Learning Era,Sequential Learning,Contemporary RecSys,Long User Histories,Reasoning in Large Language Models,Reward Hacking" | split: "," %}
{% assign guide_posts = site.posts | where_exp: "post", "post.categories contains 'AI Learning Guide'" %}
{% capture unfiled %}{% for post in guide_posts %}{% assign is_placed = false %}{% for key in placed %}{% if post.title contains key %}{% assign is_placed = true %}{% endif %}{% endfor %}{% unless is_placed %}{% include post-preview.html post=post %}{% endunless %}{% endfor %}{% endcapture %}
{% assign unfiled_trimmed = unfiled | strip %}
{% if unfiled_trimmed.size > 0 %}
## Other notes

{{ unfiled_trimmed }}
{% endif %}

<p class="listing-note">More to come.</p>

<p class="listing-note">Adjacent: <a href="/paper-readings/">Paper Readings</a> · <a href="/research-blog/">Research Blog</a></p>
