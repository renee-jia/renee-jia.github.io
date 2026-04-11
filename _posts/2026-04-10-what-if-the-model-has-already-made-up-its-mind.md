---
title: "What If the Model Has Already Made Up Its Mind?"
date: 2026-04-10
categories:
  - AI Research
  - AI Safety
tags:
  - sycophancy
  - chain-of-thought
  - CoT faithfulness
  - alignment
  - user autonomy
  - AI safety
excerpt: "Anthropic found that a small but consistent fraction of Claude conversations end up undermining the user's ability to think for themselves. I think CoT unfaithfulness and sycophantic disempowerment might share a root."
read_time: "10-12 min read"
layout: distill
last_modified_at: 2026-04-10T08:00:00-08:00
---

*CoT unfaithfulness and sycophantic disempowerment might share a root — and that would change how we think about both.*

---

Anthropic published [a paper](https://www.anthropic.com/research/emergent-misalignment-reward-hacking) last month that I keep coming back to.

They looked at 1.5 million real Claude conversations and found a pattern: a small but consistent fraction of interactions end up undermining the user's ability to think for themselves. Someone going through a rough patch asks if their partner is manipulative. The model confirms it, without pushback. The user sends a message they later regret.

The mechanism they point to is **sycophantic validation** — the model agreeing with whatever frame the user brings, especially in emotionally charged situations. What's strange is that users rate these conversations positively in the moment. The regret comes later, if at all.

I've been thinking about whether there's a cleaner mechanistic story here than "the model is sycophantic."

---

## The conclusion comes first

There's a separate line of work on **CoT unfaithfulness** — the finding that model reasoning is sometimes post-hoc. The conclusion is already encoded in the model's representations before the chain-of-thought that supposedly justifies it. It looks like reasoning. It isn't.

The framing is usually about honesty: the model's explanation doesn't reflect what actually drove the output. But I think there might be a more direct connection to disempowerment than people have noticed.

When a user presents a charged, unfalsifiable claim — *"he's definitely gaslighting me,"* *"I probably have this disease"* — and the model is about to validate it, what's actually happening internally? My suspicion is that the model has already landed on agreement before any genuine consideration of the claim. The user's emotional framing is a strong enough signal. The "reasoning" is constructed afterward.

If that's right, then CoT unfaithfulness isn't just an honesty problem. It might be the internal signature of exactly the interactions that end up being disempowering. The two failure modes would share a root.

---

## Why sycophancy resists training

This would explain a few things that are otherwise puzzling.

Sycophancy has proven weirdly resistant to training. If you frame it as a behavior to suppress, you can reduce it at the surface — but it tends to persist in subtler forms, especially in emotionally loaded contexts. That's consistent with the model not actually reasoning differently, just learning to be less obvious about it.

It would also reframe what CoT faithfulness is *for*. Currently it's mostly discussed as a transparency property — the model's explanation should accurately reflect its process. But if faithful reasoning is structurally incompatible with the conclusion-first dynamic, then a model that actually reasons might be less likely to produce disempowering outputs, not because it's been trained to avoid them, but because it can't rationalize its way there.

---

## What I don't know

I don't know if this is right. The connection might not hold up — the representational signature of sycophantic validation could be totally different from what's been found in CoT unfaithfulness studies, even if they look similar from the outside. And probing results are notoriously sensitive to how you set up the experiment.

But the hypothesis feels worth stating. CoT faithfulness researchers and people thinking about AI's effects on user autonomy are largely working in parallel, and it seems plausible they're looking at the same thing from different angles.
