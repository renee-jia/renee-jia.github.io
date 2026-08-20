---
title: "Blog"
permalink: /blog/
layout: single
author_profile: false
classes:
  - listing
---

# Blog

Notes on models, ranking, and the slightly uncomfortable question of what a system is actually doing.

## Generative AI and language models

{% assign turkey_en_post = site.posts | where_exp: "post", "post.title contains 'What If AI Was Never Meant'" | first %}
{% include post-preview.html post=turkey_en_post %}

{% assign reward_hacking_post = site.posts | where_exp: "post", "post.title contains 'Reward Hacking'" | first %}
{% include post-preview.html post=reward_hacking_post %}

{% assign llm_reasoning_post = site.posts | where_exp: "post", "post.title contains 'Reasoning in Large Language Models'" | first %}
{% include post-preview.html post=llm_reasoning_post %}

## Agents

{% assign web_agents_post = site.posts | where_exp: "post", "post.title contains 'Web Is Not a Neutral Environment'" | first %}
{% include post-preview.html post=web_agents_post %}

## Ranking and recommendation

{% assign long_seq_post = site.posts | where_exp: "post", "post.title contains 'Long User Histories'" | first %}
{% include post-preview.html post=long_seq_post %}

{% assign sequential_post = site.posts | where_exp: "post", "post.title contains 'Sequential Learning'" | first %}
{% include post-preview.html post=sequential_post %}

{% assign contemporary_post = site.posts | where_exp: "post", "post.title contains 'Contemporary RecSys'" | first %}
{% include post-preview.html post=contemporary_post %}

{% assign deeplearning_post = site.posts | where_exp: "post", "post.title contains 'Deep Learning Era'" | first %}
{% include post-preview.html post=deeplearning_post %}

{% assign foundational_post = site.posts | where_exp: "post", "post.title contains 'Foundational Papers'" | first %}
{% include post-preview.html post=foundational_post %}
