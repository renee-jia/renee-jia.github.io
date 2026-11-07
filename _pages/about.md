---
title: "About"
permalink: /about/index.html
toc: false
layout: single
author_profile: false
classes:
  - about
sidebar:
  nav: "main"
---

<div class="about-head">
  <div class="about-head__text">
    <h1>Renee Jia</h1>
    <p class="about-role">AI researcher at Meta &nbsp;|&nbsp; ex-Google</p>
    <div class="about-focus">
      <p class="about-focus__title">Learning, Memory &amp; Reasoning</p>
      <p class="about-intro">Researching how intelligent models learn from sequential experience, maintain and update knowledge over time, and use internal information for reliable reasoning.</p>
    </div>
  </div>
  <img class="about-avatar" src="{{ site.url }}{{ site.baseurl }}/assets/images/avatar.jpg" alt="Renee Jia">
</div>

## Research Interest

I study how models learn and remember, mostly through the lens of mid & post-training.

<div class="about-threads">
  <div class="about-thread">
    <h3>Learning from sequential experience</h3>
    <p>Supervised fine-tuning and what its data actually teaches, RL and reward design for multi-turn agents, continual learning without forgetting.</p>
    <p class="about-thread__links">Writing: <a href="{{ site.baseurl }}{% post_url 2026-09-09-from-memory-to-learning-continual-learning-for-llm-agents %}">From Memory to Learning</a><span class="sep">·</span><a href="{{ site.baseurl }}{% post_url 2026-09-20-every-sequence-model-is-a-memory-algorithm %}">Every Sequence Model Is a Memory Algorithm</a></p>
  </div>
  <div class="about-thread">
    <h3>Maintaining and updating knowledge over time</h3>
    <p>Memory architectures for long-horizon agents, what post-training overwrites, plasticity versus retention, moving knowledge from context into weights.</p>
    <p class="about-thread__links">Writing: <a href="{{ site.baseurl }}{% post_url 2026-07-12-distillation-in-llms-from-context-to-weights %}">Distillation in LLMs</a></p>
  </div>
  <div class="about-thread">
    <h3>Reasoning with internal information</h3>
    <p>Chain-of-thought faithfulness, shortcuts and reward hacking, metadata that silently steers behavior, evaluations that catch regressions.</p>
    <p class="about-thread__links">Writing: <a href="https://arxiv.org/abs/2609.25366">From Decorative to Load-Bearing</a><span class="sep">·</span><a href="{{ site.baseurl }}{% post_url 2026-04-10-what-if-the-model-has-already-made-up-its-mind %}">What If the Model Has Already Made Up Its Mind?</a></p>
  </div>
</div>

The same questions show up in production as user sequence modeling and personalization at scale. There the history is long, the user is a real person, and the system has far less patience than a research question does. I like working on both ends of that gap.

My path has run from research at Georgia Tech, Waterloo, and Harvard on reinforcement learning in text adventure games, to user sequence representation, teacher modeling, and distillation at Google, and now long-horizon user memory and sequence representation learning at Meta.

## Publications

<div class="pub-list">
  <p class="pub-year">2026</p>
  <ul>
    <li>
      <div class="pub-conference">
        <span class="pub-badge">TMLR 2026</span>
      </div>
      <div class="pub-details">
        <span class="pub-title"><a href="https://openreview.net/pdf?id=TiZQnKDIHq">From Decorative to Load-Bearing: Task Difficulty Shapes the Causal Role of Chain-of-Thought</a></span>
        <span class="pub-authors"><strong>Renee Jia</strong>, Di Mu</span>
        <span class="pub-venue">Transactions on Machine Learning Research (TMLR), 09/2026</span>
        <span class="pub-links"><a href="https://github.com/r2m-ai/load-bearing-cot">code</a><span class="sep">·</span><a href="https://huggingface.co/datasets/ReneeJia/cot-load-bearingness">dataset</a></span>
      </div>
    </li>
    <li>
      <div class="pub-conference">
        <span class="pub-badge">NeurIPS 2026</span>
        <span class="pub-status">accepted</span>
      </div>
      <div class="pub-details">
        <span class="pub-title">Correct Labels Are Not Enough: Reliable Metadata Can Silently Control Model Behavior</span>
        <span class="pub-authors"><strong>Renee Jia</strong>, Jinzhou Wu</span>
        <span class="pub-venue">NeurIPS 2026 Workshop: Trustworthy AI for Good (AI4Good)</span>
      </div>
    </li>
    <li>
      <div class="pub-conference">
        <span class="pub-badge">AAAI 2026</span>
        <span class="pub-status">under review</span>
      </div>
      <div class="pub-details">
        <span class="pub-title">Where Can a Language Model Change a Trade? A Decision-Transmission Audit of a Hybrid Quantitative Trading System</span>
        <span class="pub-authors"><strong>Renee Jia</strong>, Di Mu, Y. Yu, T. Liu</span>
        <span class="pub-venue">AAAI Conference on Artificial Intelligence (AAAI) 2026</span>
      </div>
    </li>
  </ul>

  <p class="pub-year">2023</p>
  <ul>
    <li>
      <div class="pub-conference">
        <span class="pub-badge">AIIDE 2023</span>
      </div>
      <div class="pub-details">
        <span class="pub-title"><a href="https://ojs.aaai.org/index.php/AIIDE/article/view/27528">Story Shaping: Teaching Agents Human-like Behavior with Stories</a></span>
        <span class="pub-authors">X. Peng, C. Cui, W. Zhou, <strong>Renee Jia</strong>, M. Riedl</span>
        <span class="pub-venue">AAAI Conference on Artificial Intelligence and Interactive Digital Entertainment (AIIDE)</span>
      </div>
    </li>
  </ul>

  <p class="pub-year">2022</p>
  <ul>
    <li>
      <div class="pub-conference">
        <span class="pub-badge">ACL 2022</span>
      </div>
      <div class="pub-details">
        <span class="pub-title"><a href="https://aclanthology.org/2022.acl-long.557/">Situated Dialogue Learning through Procedural Environment Generation</a></span>
        <span class="pub-authors">P. Ammanabrolu, <strong>Renee Jia</strong>, M. Riedl</span>
        <span class="pub-venue">Association for Computational Linguistics (ACL)</span>
      </div>
    </li>
  </ul>
</div>

<p class="pub-note">Reviewer: NeurIPS, ACL, EMNLP, EACL, NAACL, CoNLL.</p>

## Elsewhere

Off the clock: teaching snowboarding as a PSIA-AASI Level 1 instructor, and playing tournament poker ([results](https://pokerdb.thehendonmob.com/player.php?a=r&n=1103327)).

## Contact

<div class="social-icons">
  <a href="mailto:reneejia368@gmail.com">
    <i class="fas fa-fw fa-envelope-square"></i>
    <span>Email</span>
  </a>
  <a href="https://x.com/reneejjj?s=11&t=4-OvvSbKKF1hsJPSGnsoAA">
    <i class="fas fa-fw fa-xmark"></i>
    <span>X</span>
  </a>
  <a href="https://github.com/renee-jia">
    <i class="fab fa-fw fa-github"></i>
    <span>GitHub</span>
  </a>
  <a href="https://linkedin.com/in/renee-jia/">
    <i class="fab fa-fw fa-linkedin"></i>
    <span>LinkedIn</span>
  </a>
  <a href="https://renee-jia.github.io">
    <i class="fas fa-fw fa-link"></i>
    <span>Website</span>
  </a>
</div>
