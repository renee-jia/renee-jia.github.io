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

## Further notes

{% assign guide_posts = site.posts | where_exp: "post", "post.categories contains 'AI Learning Guide'" | sort: 'date' | reverse %}
{% for post in guide_posts %}
  {% unless post.title contains 'AI Beginner' %}
    {% include post-preview.html post=post %}
  {% endunless %}
{% endfor %}
