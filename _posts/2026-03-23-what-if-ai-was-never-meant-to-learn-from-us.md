---
title: "What If AI Was Never Meant to Learn From Us"
date: 2026-03-23
categories:
  - AI Research
  - Large Language Models
tags:
  - language models
  - training data
  - representation learning
  - AI philosophy
  - alignment
excerpt: "If human data isn't the optimal medium — would we even know? We've spent years feeding models everything humans have ever written, and just assumed that was the right thing to do."
read_time: "15-20 min read"
layout: distill
last_modified_at: 2026-03-23T08:00:00-08:00
---

*If human data isn't the optimal medium — would we even know?*

---

Something's been bugging me.

We've spent years feeding models everything humans have ever written — web pages, books, Reddit threads, Stack Overflow answers, basically all of it — and just assumed that was the right thing to do. More data is better. Cleaner data is better. Data that looks more like human writing is better.

But I've never seen anyone seriously ask: **is human writing actually the best format for training an intelligent system?**

Not "is it good enough." Not "how do we get more of it." Just — is it the *right thing* at all?

The question sounds almost too simple to bother with. I think that's exactly why nobody bothers.

---

## The assumption nobody ever wrote down

Math is honest about its assumptions.

Euclid wrote his postulates out explicitly. "A straight line can be drawn between any two points." He didn't prove it — he just said, I'm taking this as given, everything else follows. Because he was explicit, people later could actually question it. Riemann eventually swapped one out and got a completely different geometry — which turned out to be the one that describes how space actually works.

The point isn't that Euclid was wrong. It's that writing the assumption down is what made it possible to challenge.

Our whole AI training setup has an assumption baked in that nobody ever wrote down:

> **Human data is the natural way to train intelligent systems.**

It's just there. In every paper about data quality, every discussion about scale, every training pipeline. Nobody argues for it. Nobody even notices it. It's so deeply assumed it doesn't register as an assumption.

So let's write it down. And then ask if it's actually true.

---

## Something weird is going on inside these models

In early 2025, Anthropic put out a paper I've read a few times. They figured out how to trace the actual path a model takes when it solves something — not rough guesses from attention weights, but real step-by-step chains: this input activates these features, those features trigger this computation, that's how you get the output.

What they found is genuinely strange.

**The model doesn't know how it does math.**

Ask Claude to add $36 + 47$, then ask how it did it. It'll explain carrying digits, adding columns — the standard grade school method.

That's not what's actually happening. The tracing shows two things running in parallel: one getting a rough estimate ("somewhere around 80"), one working out the exact units digit ($6+7=13$). They combine at the end to give the answer.

What the model says it's doing and what it's actually doing are two different things. And the model has no idea. Its explanation isn't a lie — it's just a story told in human-readable language about a process it can't actually see.

**There's a layer of thinking that happens before language.**

The cross-language finding is even more interesting.

When the model works with the concept "small" — whether the input is English *small*, French *petit*, or Chinese *小* — the same internal features activate. There's a language-neutral layer where the actual reasoning happens. Input comes in, gets processed there, then gets translated into whatever language you asked in at the very end.

Language isn't where the thinking happens. It's just the input and output format.

And bigger models do this *more*. As models get larger, they lean harder on this abstract layer and less on any specific language. Nobody designed this. It just showed up.

This isn't an answer to anything. It's a crack — a hint that the model's natural way of representing things might not be human language at all.

---

## Five ways human data is broken as training material

Human data comes from human minds with human limitations. Those limitations don't go away when you use the data to train something else. They get quietly baked in everywhere.

Here are the five that bother me most.

### 1. We already flattened our thoughts before writing them down

Thinking is probably parallel. You hold multiple threads at once, make connections across different things simultaneously. Writing is one word after another. By the time a thought makes it onto a page, it's already been squashed into a line.

Models aren't learning from thought. They're learning from the already-flattened record of it.

And here's the weird part: the transformer architecture can actually process things in parallel. But the training goal — predict the next token — rewards it for rebuilding linear sequences. There's a real mismatch between what the architecture could do and what the training is actually asking for.

### 2. Words mean too many things

One token, potentially dozens of meanings, figured out from context. A huge chunk of training is basically teaching the model that the same word can mean completely different things — which from a learning standpoint is just noise.

Most of the ambiguity isn't even obvious. "Fair" in a legal document means something different than in a casual conversation. "Significant" in a stats paper is a technical term. "Interesting" online is usually sarcasm. They all look the same in the raw data.

### 3. Cultural assumptions are hidden everywhere

Human data isn't neutral. It carries history, power structures, and cultural frames — all as invisible defaults.

Not the obvious stuff like demographic bias, which at least has a name. I mean deeper things: what makes a good argument, whether time is linear, where the individual ends and the group begins. These vary a lot across cultures. In the training data, one set of answers got overrepresented enough to become the default, and nobody labeled it as such.

Models don't just learn facts. They learn a hidden framework for how facts are supposed to fit together. That framework came from somewhere specific.

### 4. Almost nothing in human writing explores what doesn't exist

Most of human writing is about things that actually happened or actually exist. Systematically exploring counterfactuals — what if this had been different, what are all the possible states here, what happens if you remove this constraint — is rare. And when it shows up, it's usually telling a story, not genuinely mapping out a space of possibilities.

But a lot of useful reasoning is counterfactual. Debugging is. Risk assessment is. Forming a hypothesis is. The training data is thinnest exactly where this matters most.

### 5. Nobody writes down how they actually figured things out

This one bothers me the most.

Every paper shows the result. Every book shows the conclusion. Every tutorial shows the working solution. Almost completely missing: the actual process — the wrong turns, the confusion, the slow movement from "I don't get this" to "oh, I see."

Models get trained on the endpoints of human thinking. Not on thinking itself. So they learn to produce things that look like answers — without ever being trained on what it looks like to work through something you genuinely don't understand yet.

---

## So what would better training look like

Thought experiment. Forget what's practical. If you could redesign training from scratch with one goal — let models find whatever structure works best *for them* — what would you try?

**Train directly in representation space.** Right now, representations are a side effect of predicting text. But what if you could specify what good representations should look like — which concepts should cluster, what relationships should be linear — and optimize toward that directly? Make the structure itself the goal, instead of hoping it shows up on its own.

**Take away the human playbook.** AlphaZero didn't learn Go from human games. It played against itself with just the rules, developed a style completely unlike how humans play, and beat the best players in the world. Language doesn't have a clear win condition the way Go does. But things near language might — is this proof valid, does this code run, does this simulation conserve energy. If models trained mainly on tasks with objective answers and figured out their own representations for solving them, the result might look very different from anything we have now.

**Use other data types to get around language.** Images, audio, physics simulations — none of these come from human language. They come from how the world actually is. Training on them might not just mean more data. It might mean a way around some of the distortions that come from using language as the medium.

**The weird one: models that know where they're confused.** What if a model could look at its own internal representations and find the spots that seem wrong — a concept that should be smooth but has a break in it, two ideas that should be separate but are tangled — and flag those as places needing better training? That would be metacognition that's actually useful. Not "I don't know X," but "my understanding of X is broken in this specific way." We don't have this. But it feels less like something that's impossible and more like something nobody's tried to build.

---

## The turkey

There's a story from *The Three-Body Problem* I keep coming back to.

A turkey gets fed every morning at nine. It watches this happen a thousand times. It's careful — tracks the pattern, checks it across seasons, builds up real confidence. Its conclusion: food comes at nine every morning.

Works perfectly. Until the night before Thanksgiving.

---

The turkey didn't fail because it reasoned badly. It failed because nothing in its observation window could tell it what was outside. The farmer's actual intentions were never in the data. The turkey had no way to even know there was something it was missing.

Now: us.

We train models on human data. We test them with human benchmarks. We judge whether they're improving by human ideas of what "better" looks like — scores, preference ratings, how it feels to use them. The whole system holds together. Within this window, progress is real.

But what's outside the window?

If there's a fundamentally better way for these systems to learn — one that doesn't go through human language, doesn't optimize for human preferences, uses representations that don't come from predicting text — we'd have basically no way to find it. Our evals wouldn't catch it. Our loss functions wouldn't point to it. Our intuitions about "better" would actively pull us away from it.

---

If the best possible learning path for these models lies outside what human data covers, we may never know.

Not because we're not smart enough or not trying hard enough. Because every tool we use to check is built on the same invisible assumption. We define "understanding" in human language. We define "progress" with human intuition. We define "aligned" through human preference. The whole thing holds together — exactly as well as the turkey's thousand-morning dataset did.

Holding together has never meant being right. It just means one thing: **you haven't met Thanksgiving yet.**

---

We're building a taller and taller tower on a foundation nobody explicitly checked. The taller it gets, the more confident we feel — because look, it's still standing. All these new capabilities, all these benchmarks, all this progress.

Euclid's parallel postulate felt that solid too. For two thousand years.

---

> Maybe we are working very hard to train a more and more perfect turkey.
>
> We don't know what Thanksgiving is.
>
> We don't even know that we don't know.
