---
title: "Distillation in LLMs: From Context to Weights"
date: 2026-07-12
research_kind: open-problem
categories:
  - Research Blog
  - AI Research
tags:
  - knowledge distillation
  - in-context learning
  - context distillation
  - on-policy distillation
  - self-distillation
  - continual learning
  - test-time training
excerpt: "A model that reads three examples and nails the fourth has learned something, and its weights didn't move. Where is that lesson, and how do you keep it? Follow the question and distillation stops being a compression trick: the teacher doesn't have to be bigger, it just has to have seen more context."
read_time: "10-12 min read"
layout: distill
toc: false
last_modified_at: 2026-07-12T08:00:00-08:00
---

*What in-context learning teaches, and how to keep it after the context is gone.*

---

Language models are good at learning from context.

Give one a few examples and it follows the format. Give it a long document and it answers questions about facts it has never seen. Give it a conversation history and it adapts to what happened ten turns ago. None of this touches the weights. The model learns something, and the parameters sit still.

Take that literally for a second. Something was learned. Where is it?

Not in the weights. Not really in the context either: the context is the input, not the lesson. The lesson is whatever the model now does differently because of that input. And a lesson that lives nowhere in particular doesn't last. Close the window and it's gone.

So the question is how to keep it. In another model, in a compact memory, or in the same model's own weights, so it outlives the context that taught it.

That's a distillation question, even though it doesn't look like one. Distillation usually means squeezing a big model into a small one. But strip it down and distillation is just this: a teacher knows something, a student doesn't, and you train the student to act like the teacher. Nothing there says the teacher has to be bigger. **It only has to know more, and having seen more context is the cheapest way to know more.**

---

## The lesson is a shift, not a vector

Sequence modeling usually asks how tokens become hidden states. Context goes in, a vector comes out, and the vector is what the model knows.

That view is fine as far as it goes. Show a model three examples of an unfamiliar rewrite rule and it applies the rule to a fourth input. The interesting thing isn't the embedding of those three examples. It's the shift in what the model predicts.

Write the model as

$$
p_\theta(y \mid q, C),
$$

with $q$ the query, $y$ the output, and $C$ everything else in the window: instructions, demonstrations, retrieved passages, tool outputs, earlier turns. You can ask how $C$ gets encoded. Or you can ask how $C$ moves $p_\theta(y \mid q)$ to $p_\theta(y \mid q, C)$. Same model, same weights, two different objects.

<figure class="post-figure">
<svg viewBox="0 0 660 250" xmlns="http://www.w3.org/2000/svg" role="img" aria-label="Two views of in-context learning: the representation view encodes the context into hidden states; the adaptation view tracks how the output distribution shifts once the context is present">
<defs>
<marker id="f1a" markerWidth="7" markerHeight="7" refX="6" refY="3" orient="auto"><path d="M0,0 L6,3 L0,6 z" fill="#9a7048"/></marker>
<marker id="f1b" markerWidth="7" markerHeight="7" refX="6" refY="3" orient="auto"><path d="M0,0 L6,3 L0,6 z" fill="#6b7a52"/></marker>
</defs>
<style>.lb{font-family:'Source Sans 3',system-ui,sans-serif;font-size:10.5px;fill:#8a7b6b}.lbs{font-family:'Source Sans 3',system-ui,sans-serif;font-size:11px;fill:#3d342b}.hd{font-family:'Source Sans 3',system-ui,sans-serif;font-size:11.5px;fill:#3d342b;font-weight:600}</style>

<text x="20" y="28" class="hd">Representation view</text>
<text x="20" y="44" class="lb">what gets encoded</text>
<rect x="20" y="70" width="110" height="60" rx="4" fill="#f3ebe0" stroke="#e4d5c4"/>
<text x="75" y="95" class="lbs" text-anchor="middle">context C</text>
<text x="75" y="111" class="lb" text-anchor="middle">3 demos + query</text>
<path d="M 132 100 H 168" stroke="#9a7048" stroke-width="1.3" fill="none" marker-end="url(#f1a)"/>
<rect x="172" y="70" width="110" height="60" rx="4" fill="#f0e6d6" stroke="#9a7048"/>
<text x="227" y="95" class="lbs" text-anchor="middle">hidden states</text>
<text x="227" y="111" class="lb" text-anchor="middle">h(C)</text>
<text x="151" y="190" class="lb" text-anchor="middle">the object is a vector</text>

<line x1="330" y1="20" x2="330" y2="230" stroke="#e4d5c4" stroke-dasharray="3 3"/>

<text x="360" y="28" class="hd">Adaptation view</text>
<text x="360" y="44" class="lb">what changes</text>

<text x="400" y="66" class="lb" text-anchor="middle">p(y | q)</text>
<rect x="364" y="112" width="12" height="18" fill="#e4d5c4"/>
<rect x="380" y="100" width="12" height="30" fill="#e4d5c4"/>
<rect x="396" y="106" width="12" height="24" fill="#e4d5c4"/>
<rect x="412" y="96" width="12" height="34" fill="#e4d5c4"/>
<rect x="428" y="110" width="12" height="20" fill="#e4d5c4"/>
<line x1="360" y1="130" x2="444" y2="130" stroke="#8a7b6b" stroke-width="1"/>

<path d="M 460 100 H 500" stroke="#6b7a52" stroke-width="1.3" fill="none" marker-end="url(#f1b)"/>
<text x="480" y="92" class="lb" text-anchor="middle">+ C</text>

<text x="560" y="66" class="lb" text-anchor="middle">p(y | q, C)</text>
<rect x="524" y="124" width="12" height="6" fill="#e4d5c4"/>
<rect x="540" y="120" width="12" height="10" fill="#e4d5c4"/>
<rect x="556" y="72" width="12" height="58" fill="#6b7a52"/>
<rect x="572" y="122" width="12" height="8" fill="#e4d5c4"/>
<rect x="588" y="126" width="12" height="4" fill="#e4d5c4"/>
<line x1="520" y1="130" x2="604" y2="130" stroke="#8a7b6b" stroke-width="1"/>

<text x="484" y="190" class="lb" text-anchor="middle">the object is a shift in behavior</text>
<text x="484" y="206" class="lb" text-anchor="middle">same weights before and after</text>
</svg>
<figcaption>Same model, same weights. The representation view asks what the context became inside the network. The adaptation view asks what the context did to the predictions. Distillation has traditionally lived on the left. The rest of this post argues it belongs on the right.</figcaption>
</figure>

The second one doesn't need any parameter to change. In-context learning moves behavior purely through conditioning. Some people argue the forward pass is implicitly running gradient descent on the demonstrations. That's a nice way of saying the shift is real enough to have its own dynamics. But you don't need that result. If you wanted to copy something out of a context-conditioned model, the shift is the thing to copy.

That also says what kind of distillation this is. Classic distillation copies the teacher's internals: hidden states, attention maps, or at the loosest, its output distribution. All of that copies a state. Here the target isn't a state. It's a response to information: how outputs move when context arrives. Huang and colleagues tried an early version in 2022, distilling a big model's few-shot ability into a small one rather than any particular task. The simplest version predates it.

---

## A prompt can be turned into weights

In-context adaptation is temporary. Take the demonstrations out of the window and the behavior goes with them. Context distillation makes it stick.

The whole setup fits in two rows:

| | Sees |
|---|---|
| **Teacher** | task + instructions + demonstrations |
| **Student** | task only |

Same model on both sides. The teacher predicts with the extra context. The student is trained to match those predictions without it. Afterwards the student does what the prompt used to make it do, and the prompt is gone.

<figure class="post-figure">
<svg viewBox="0 0 660 230" xmlns="http://www.w3.org/2000/svg" role="img" aria-label="Context distillation: a teacher that sees instructions and demonstrations plus the task produces a target distribution; a student that sees only the task is trained to match it">
<defs>
<marker id="f3a" markerWidth="7" markerHeight="7" refX="6" refY="3" orient="auto"><path d="M0,0 L6,3 L0,6 z" fill="#9a7048"/></marker>
<marker id="f3r" markerWidth="7" markerHeight="7" refX="6" refY="3" orient="auto"><path d="M0,0 L6,3 L0,6 z" fill="#a8543f"/></marker>
</defs>
<style>.lb{font-family:'Source Sans 3',system-ui,sans-serif;font-size:10.5px;fill:#8a7b6b}.lbs{font-family:'Source Sans 3',system-ui,sans-serif;font-size:11px;fill:#3d342b}.hd{font-family:'Source Sans 3',system-ui,sans-serif;font-size:11.5px;fill:#3d342b;font-weight:600}</style>

<text x="20" y="40" class="hd">Teacher</text>
<rect x="90" y="24" width="90" height="24" rx="3" fill="#e6e9dc" stroke="#6b7a52"/><text x="135" y="40" class="lbs" text-anchor="middle">instructions</text>
<rect x="186" y="24" width="90" height="24" rx="3" fill="#e6e9dc" stroke="#6b7a52"/><text x="231" y="40" class="lbs" text-anchor="middle">demos</text>
<rect x="282" y="24" width="90" height="24" rx="3" fill="#f3ebe0" stroke="#e4d5c4"/><text x="327" y="40" class="lbs" text-anchor="middle">task</text>
<path d="M 376 36 H 410" stroke="#9a7048" stroke-width="1.3" fill="none" marker-end="url(#f3a)"/>
<rect x="414" y="20" width="72" height="32" rx="4" fill="#f0e6d6" stroke="#9a7048"/><text x="450" y="40" class="lbs" text-anchor="middle">model θ</text>
<path d="M 490 36 H 524" stroke="#9a7048" stroke-width="1.3" fill="none" marker-end="url(#f3a)"/>
<text x="536" y="40" class="lbs">p(y | task, ctx)</text>

<text x="20" y="130" class="hd">Student</text>
<rect x="282" y="114" width="90" height="24" rx="3" fill="#f3ebe0" stroke="#e4d5c4"/><text x="327" y="130" class="lbs" text-anchor="middle">task</text>
<rect x="90" y="114" width="186" height="24" rx="3" fill="none" stroke="#e4d5c4" stroke-dasharray="3 3"/><text x="183" y="130" class="lb" text-anchor="middle">(nothing here)</text>
<path d="M 376 126 H 410" stroke="#9a7048" stroke-width="1.3" fill="none" marker-end="url(#f3a)"/>
<rect x="414" y="110" width="72" height="32" rx="4" fill="#f0e6d6" stroke="#9a7048"/><text x="450" y="130" class="lbs" text-anchor="middle">model θ'</text>
<path d="M 490 126 H 524" stroke="#9a7048" stroke-width="1.3" fill="none" marker-end="url(#f3a)"/>
<text x="536" y="130" class="lbs">p(y | task)</text>

<path d="M 585 56 V 108" stroke="#a8543f" stroke-width="1.3" fill="none" marker-end="url(#f3r)"/>
<text x="596" y="86" class="lb" fill="#a8543f">loss, pushes θ'</text>

<text x="20" y="190" class="lb">after training: the student handles the task bare, and the instructions and demos are now in θ'</text>
<text x="20" y="206" class="lb">what moved isn't a vector. It's the behavior the context used to induce.</text>
</svg>
<figcaption>Context distillation. The teacher and student share weights at the start. The only asymmetry is what they're allowed to see. Training closes the gap, and what used to be a prompt becomes a parameter change.</figcaption>
</figure>

Snell, Klein and Zhong did this in 2022 for instructions, step-by-step scratchpads and few-shot examples. Their student is literally the same model, fine-tuned on its own context-conditioned outputs. A year earlier Askell and colleagues at Anthropic had used a KL version of the same trick. They baked a long alignment prompt into the weights so it didn't have to ride along on every call. Same move both times. A capability the model only had with the right context becomes one it has without it.

One limit, though. A single static student can't absorb every possible context. If two contexts contradict each other, the student has to pick. If a context never showed up in training, the student has no reason to generalize to it. So this isn't "put all prompts into the weights." It's "learn the behaviors a known distribution of contexts induces." Smaller promise, and one you can keep.

It also leaves something open. The student trains on some set of sequences. Whose?

---

## Grade the student's own mistakes

Supervised fine-tuning imitates target responses with negative log-likelihood. Swap the human-written targets for teacher-generated ones and you have hard-label distillation. Match the teacher's full next-token distribution instead of a single sampled token and you have soft-label distillation, which carries more information per position.

That's the supervision axis. There's a second axis that gets less attention, and it decides whether any of this works at inference time: *whose* sequences the loss is computed on.

- **Off-policy:** fixed or teacher-generated sequences. The student is corrected on prefixes the teacher produced.
- **On-policy:** student-generated sequences. The student is corrected on prefixes it actually produces.

Generation is sequential, and that's the whole problem. At some point the student samples a token the teacher wouldn't have. From then on, every prediction is conditioned on a history the training data never covered. Off-policy distillation tells the student what to do along the teacher's path. It says nothing about what to do once the student has left it, and the student always leaves it eventually.

<figure class="post-figure">
<svg viewBox="0 0 660 260" xmlns="http://www.w3.org/2000/svg" role="img" aria-label="Off-policy versus on-policy distillation: the student's trajectory diverges from the teacher's at a token; off-policy supervision only covers the teacher's path, on-policy supervision follows the student's path with teacher feedback at each step">
<defs>
<marker id="f4g" markerWidth="7" markerHeight="7" refX="6" refY="3" orient="auto"><path d="M0,0 L6,3 L0,6 z" fill="#6b7a52"/></marker>
<marker id="f4r" markerWidth="7" markerHeight="7" refX="6" refY="3" orient="auto"><path d="M0,0 L6,3 L0,6 z" fill="#a8543f"/></marker>
</defs>
<style>.lb{font-family:'Source Sans 3',system-ui,sans-serif;font-size:10.5px;fill:#8a7b6b}.lbs{font-family:'Source Sans 3',system-ui,sans-serif;font-size:11px;fill:#3d342b}.hd{font-family:'Source Sans 3',system-ui,sans-serif;font-size:11.5px;fill:#3d342b;font-weight:600}</style>

<text x="20" y="28" class="hd">Off-policy</text>
<text x="20" y="44" class="lb">loss lives on the teacher's sequence</text>
<g>
<circle cx="40" cy="90" r="6" fill="#6b7a52"/><circle cx="90" cy="90" r="6" fill="#6b7a52"/><circle cx="140" cy="90" r="6" fill="#6b7a52"/><circle cx="190" cy="90" r="6" fill="#6b7a52"/><circle cx="240" cy="90" r="6" fill="#6b7a52"/><circle cx="290" cy="90" r="6" fill="#6b7a52"/>
<line x1="46" y1="90" x2="84" y2="90" stroke="#6b7a52" stroke-width="1.3"/><line x1="96" y1="90" x2="134" y2="90" stroke="#6b7a52" stroke-width="1.3"/><line x1="146" y1="90" x2="184" y2="90" stroke="#6b7a52" stroke-width="1.3"/><line x1="196" y1="90" x2="234" y2="90" stroke="#6b7a52" stroke-width="1.3"/><line x1="246" y1="90" x2="284" y2="90" stroke="#6b7a52" stroke-width="1.3"/>
<text x="40" y="74" class="lb" text-anchor="middle">✓</text><text x="90" y="74" class="lb" text-anchor="middle">✓</text><text x="140" y="74" class="lb" text-anchor="middle">✓</text><text x="190" y="74" class="lb" text-anchor="middle">✓</text><text x="240" y="74" class="lb" text-anchor="middle">✓</text><text x="290" y="74" class="lb" text-anchor="middle">✓</text>
<path d="M 140 96 C 160 120, 180 140, 200 146" stroke="#a8543f" stroke-width="1.3" fill="none" stroke-dasharray="3 3" marker-end="url(#f4r)"/>
<circle cx="214" cy="150" r="6" fill="none" stroke="#a8543f" stroke-width="1.3"/><circle cx="264" cy="158" r="6" fill="none" stroke="#a8543f" stroke-width="1.3"/>
<line x1="220" y1="151" x2="258" y2="157" stroke="#a8543f" stroke-width="1.3" stroke-dasharray="3 3"/>
<text x="150" y="128" class="lb" fill="#a8543f">student samples</text>
<text x="150" y="141" class="lb" fill="#a8543f">a different token</text>
<text x="214" y="182" class="lb" text-anchor="middle">no supervision out here</text>
</g>

<line x1="330" y1="20" x2="330" y2="240" stroke="#e4d5c4" stroke-dasharray="3 3"/>

<text x="360" y="28" class="hd">On-policy</text>
<text x="360" y="44" class="lb">loss lives on the student's sequence</text>
<g>
<circle cx="380" cy="90" r="6" fill="#6b7a52"/><circle cx="430" cy="90" r="6" fill="#6b7a52"/><circle cx="480" cy="90" r="6" fill="#6b7a52"/>
<line x1="386" y1="90" x2="424" y2="90" stroke="#6b7a52" stroke-width="1.3"/><line x1="436" y1="90" x2="474" y2="90" stroke="#6b7a52" stroke-width="1.3"/>
<path d="M 480 96 C 500 120, 520 140, 540 146" stroke="#a8543f" stroke-width="1.3" fill="none"/>
<circle cx="552" cy="150" r="6" fill="#a8543f"/><circle cx="602" cy="158" r="6" fill="#a8543f"/>
<line x1="558" y1="151" x2="596" y2="157" stroke="#a8543f" stroke-width="1.3"/>
<path d="M 552 118 V 140" stroke="#6b7a52" stroke-width="1.3" fill="none" marker-end="url(#f4g)"/>
<path d="M 602 126 V 148" stroke="#6b7a52" stroke-width="1.3" fill="none" marker-end="url(#f4g)"/>
<text x="552" y="112" class="lb" text-anchor="middle">teacher</text>
<text x="602" y="120" class="lb" text-anchor="middle">teacher</text>
<text x="520" y="182" class="lb" text-anchor="middle">the teacher grades the student's own prefix,</text>
<text x="520" y="196" class="lb" text-anchor="middle">including the part after it went wrong</text>
</g>
<text x="330" y="232" class="lb" text-anchor="middle">same teacher, same loss, different states it gets applied to</text>
</svg>
<figcaption>The compounding-error problem. Off-policy training corrects the student only on prefixes the teacher wrote. On-policy training samples from the student and asks the teacher what it would have done at each of the student's own positions, including after the first mistake.</figcaption>
</figure>

So train on the student's own sequences. Let the student generate, then have the teacher score each position of what the student actually wrote. Agarwal and colleagues worked this out as Generalized Knowledge Distillation, with a choice of divergence to go with it. Thinking Machines published a plainer version of the same recipe last October: the student samples, the teacher assigns a reverse KL at every token. Every position gets a signal, where reinforcement learning hands back one number per episode. They argue it's a big part of why the recipe is so much cheaper than RL.

There's a side benefit. In a different paper, RL's Razor, the same MIT group found that forgetting tracks how far an update drifts from the starting model. On-policy training drifts less than SFT. That paper was about RL, but the argument is about on-policy data, not about rewards. Ye and colleagues' on-policy context distillation this year is the bet that it carries over.

So now the teacher is a context-conditioned model, and it grades the bare model's own attempts. Notice what that sentence doesn't need: a second model.

---

## The teacher can be the same model

What if teacher and student are literally the same weights?

Taken naively, that's a no-op. A model matching its own predictions under identical conditions gets a zero gradient. Self-distillation only works if there's an asymmetry, and there are a few to choose from. The teacher sees a demonstration the student doesn't. The teacher gets more inference compute, more samples, more search. The teacher is a later checkpoint. Any of these breaks the tie, and context is the cheapest one to arrange.

Self-Distillation Fine-Tuning, from Shenfeld and colleagues this January, is the two previous ideas stacked. The teacher is the model with an expert demonstration in context. The student is the same model without it. The student generates its own attempts, and the demonstration-conditioned teacher scores every token of them. The teacher's weights are a slow moving average of the student's, so the two never drift apart. Context supplies the asymmetry. On-policy sampling supplies the data. The model's own in-context ability is the teacher, and what it teaches ends up in the weights.

<figure class="post-figure">
<svg viewBox="0 0 660 230" xmlns="http://www.w3.org/2000/svg" role="img" aria-label="Self-distillation as a loop: the same model, once with a demonstration in context and once without, with the demonstration-conditioned copy supervising the bare copy's own samples">
<defs>
<marker id="f5a" markerWidth="7" markerHeight="7" refX="6" refY="3" orient="auto"><path d="M0,0 L6,3 L0,6 z" fill="#9a7048"/></marker>
<marker id="f5g" markerWidth="7" markerHeight="7" refX="6" refY="3" orient="auto"><path d="M0,0 L6,3 L0,6 z" fill="#6b7a52"/></marker>
<marker id="f5r" markerWidth="7" markerHeight="7" refX="6" refY="3" orient="auto"><path d="M0,0 L6,3 L0,6 z" fill="#a8543f"/></marker>
</defs>
<style>.lb{font-family:'Source Sans 3',system-ui,sans-serif;font-size:10.5px;fill:#8a7b6b}.lbs{font-family:'Source Sans 3',system-ui,sans-serif;font-size:11px;fill:#3d342b}.hd{font-family:'Source Sans 3',system-ui,sans-serif;font-size:11.5px;fill:#3d342b;font-weight:600}</style>

<rect x="250" y="20" width="160" height="40" rx="4" fill="#f0e6d6" stroke="#9a7048"/>
<text x="330" y="38" class="hd" text-anchor="middle">one model, θ</text>
<text x="330" y="53" class="lb" text-anchor="middle">same weights on both sides</text>

<path d="M 290 62 C 250 80, 180 90, 150 104" stroke="#9a7048" stroke-width="1.3" fill="none" marker-end="url(#f5a)"/>
<path d="M 370 62 C 410 80, 480 90, 510 104" stroke="#9a7048" stroke-width="1.3" fill="none" marker-end="url(#f5a)"/>

<rect x="40" y="108" width="200" height="60" rx="4" fill="#e6e9dc" stroke="#6b7a52"/>
<text x="140" y="128" class="hd" text-anchor="middle">teacher role</text>
<text x="140" y="144" class="lb" text-anchor="middle">task + expert demonstration in context</text>
<text x="140" y="158" class="lb" text-anchor="middle">knows more, only for now</text>

<rect x="420" y="108" width="200" height="60" rx="4" fill="#f3ebe0" stroke="#e4d5c4"/>
<text x="520" y="128" class="hd" text-anchor="middle">student role</text>
<text x="520" y="144" class="lb" text-anchor="middle">task only, samples its own attempts</text>
<text x="520" y="158" class="lb" text-anchor="middle">knows less, but gets to keep what it learns</text>

<path d="M 420 150 H 250" stroke="#a8543f" stroke-width="1.3" fill="none" marker-end="url(#f5r)"/>
<text x="335" y="144" class="lb" text-anchor="middle" fill="#a8543f">student's samples</text>
<path d="M 240 160 H 410" stroke="#6b7a52" stroke-width="1.3" fill="none" marker-end="url(#f5g)"/>
<text x="330" y="176" class="lb" text-anchor="middle">per-token targets from the teacher role</text>

<path d="M 622 138 C 654 138, 654 40, 414 40" stroke="#9a7048" stroke-width="1" fill="none" stroke-dasharray="3 3" marker-end="url(#f5a)"/>
<text x="330" y="206" class="lb" text-anchor="middle">the update changes θ, so next round the teacher role is better too</text>
</svg>
<figcaption>Self-distillation with a context asymmetry. Nothing here needs a second, larger model. The teacher is the student plus a demonstration, and the demonstration's effect gets written into the weights by supervising the student's own trajectories.</figcaption>
</figure>

The numbers are better than you'd hope. On the paper's skill-learning tasks with a 7B instruct model, it beats SFT on the new skill, and it keeps the old capabilities close to where the base model had them. SFT gives up about ten points of those.

| Qwen2.5-7B-Instruct | New skill, SFT → SDFT | Prior capabilities, SFT → SDFT |
|---|---|---|
| Science Q&A | 66.2 → 70.2 | 53.4 → 64.5 |
| Tool use | 63.2 → 70.6 | 56.0 → 65.4 |

The base model's prior-capability average is 65.5, so the right column is roughly the gap between forgetting and not. Whether that holds outside their settings is open; the gain shrinks at 3B, and the sequential-learning results are figures rather than tables. But the structural point doesn't depend on the numbers. Knowledge transfer doesn't need two independently trained models. It can run between two informational states of one model, and the context window is what makes those states different.

All of that is the clean case, where context is a fixed block of text and the lesson is a task. Two things fall outside it.

---

## Two things that don't transfer yet

The first is the update rule. Long-running agents, persistent assistants, streaming sequence models and test-time-training layers don't see a context once. They keep an internal state and keep updating it. For those, there's a difference between what the memory holds and how it changes:

> **Distilling a memory representation is not the same as distilling the mechanism that updates it.**

You can train a student to land on the teacher's final memory state for one sequence. That says almost nothing about the next sequence. If the student got there by a different update rule, the two diverge the moment the input changes. Matching endpoints copies a state. Matching the update is the thing that would actually carry over.

<figure class="post-figure">
<svg viewBox="0 0 660 250" xmlns="http://www.w3.org/2000/svg" role="img" aria-label="Two panels: on the training sequence the student's memory state ends where the teacher's does; on a new sequence the two trajectories diverge because the update rules differ">
<style>.lb{font-family:'Source Sans 3',system-ui,sans-serif;font-size:10.5px;fill:#8a7b6b}.lbs{font-family:'Source Sans 3',system-ui,sans-serif;font-size:11px;fill:#3d342b}.hd{font-family:'Source Sans 3',system-ui,sans-serif;font-size:11.5px;fill:#3d342b;font-weight:600}</style>

<text x="20" y="28" class="hd">Sequence seen in training</text>
<text x="20" y="44" class="lb">final states match</text>
<line x1="30" y1="190" x2="300" y2="190" stroke="#e4d5c4"/>
<line x1="30" y1="190" x2="30" y2="60" stroke="#e4d5c4"/>
<text x="300" y="204" class="lb" text-anchor="end">t</text>
<text x="24" y="66" class="lb" text-anchor="end">M</text>
<path d="M 30 170 C 90 110, 150 150, 200 120 S 270 100, 290 96" stroke="#6b7a52" stroke-width="1.6" fill="none"/>
<path d="M 30 170 C 70 160, 130 90, 190 110 S 260 110, 290 96" stroke="#a8543f" stroke-width="1.6" fill="none" stroke-dasharray="4 3"/>
<circle cx="290" cy="96" r="5" fill="#9a7048"/>
<text x="282" y="86" class="lb" text-anchor="end">same endpoint</text>

<line x1="330" y1="20" x2="330" y2="230" stroke="#e4d5c4" stroke-dasharray="3 3"/>

<text x="360" y="28" class="hd">New sequence</text>
<text x="360" y="44" class="lb">same start, different rule, different place</text>
<line x1="370" y1="190" x2="640" y2="190" stroke="#e4d5c4"/>
<line x1="370" y1="190" x2="370" y2="60" stroke="#e4d5c4"/>
<text x="640" y="204" class="lb" text-anchor="end">t</text>
<text x="364" y="66" class="lb" text-anchor="end">M</text>
<path d="M 370 170 C 420 150, 470 140, 520 110 S 600 80, 630 70" stroke="#6b7a52" stroke-width="1.6" fill="none"/>
<path d="M 370 170 C 420 165, 470 170, 520 150 S 600 150, 630 160" stroke="#a8543f" stroke-width="1.6" fill="none" stroke-dasharray="4 3"/>
<circle cx="630" cy="70" r="5" fill="#6b7a52"/>
<circle cx="630" cy="160" r="5" fill="#a8543f"/>

<line x1="370" y1="226" x2="394" y2="226" stroke="#6b7a52" stroke-width="1.6"/><text x="400" y="230" class="lb">teacher's memory</text>
<line x1="500" y1="226" x2="524" y2="226" stroke="#a8543f" stroke-width="1.6" stroke-dasharray="4 3"/><text x="530" y="230" class="lb">student's memory</text>
</svg>
<figcaption>Endpoint matching versus dynamics matching. Left: the student reproduces the teacher's final state on the sequence it was trained on. Right: on a fresh stream the two update rules take the state to different places. What needed to transfer was the rule.</figcaption>
</figure>

Which update rule depends on the architecture. Gated recurrent memories decay and gate. Gated DeltaNets apply an error-correcting write. Test-time-training layers take a gradient step on a small inner model. [An earlier post](/research%20blog/ai%20research/every-sequence-model-is-a-memory-algorithm/) read all of these as online learning algorithms for their own memory, and that reading is what makes the question sharp. If the memory is a learner, distilling the memory means distilling the learner. A loss on final states won't do it. Whether anything will is, as far as I can tell, open.

The second is interference. Writing a lesson into the weights means writing it into a shared resource, and every lesson that lands there can disturb the ones already present. Which is why there are four places to keep a lesson rather than one, and why none of them wins outright:

| Where it lives | What it's good at | What it costs |
|---|---|---|
| Context window | Flexible; nothing is lost while it's there | Temporary; expensive at scale |
| Compressed external memory | Cheaper to store and to read | Compression decides what's worth keeping, and can be wrong |
| Online-updated latent state | Adapts continuously as the stream arrives | The update dynamics are hard to transfer or inspect |
| Model parameters | Persistent; free at inference time | Can interfere with what the model already knew |

The last row is continual learning, and self-distillation is the most promising route into it I've seen. But the table is also the reason it won't be the whole answer. The question isn't how to max out plasticity or retention. It's when a lesson should move from one row to the next.

---

## Maybe worth asking

The textbook question about distillation is how a small model can imitate a large one. It's a good question and a narrow one. The version that's been on my mind is wider:

> **How does behavior learned in one context get moved into another model, a memory, or a set of weights?**

Reading in-context learning as a shift rather than an embedding is what connects the pieces. Context distillation moves the shift into weights. On-policy distillation makes the move survive the student's own mistakes. Self-distillation shows the teacher can be the same model with more information. Together they're one procedure, not three papers.

What I keep coming back to is what that procedure doesn't carry. It moves *what* a model learned from context. The harder version asks whether you can move *how* it learned, so a student picks up not just the teacher's answers under new evidence but its way of changing its mind. Can we distill not only what models know, but how they learn?

---

## Notes

- Hinton, Vinyals & Dean, *Distilling the Knowledge in a Neural Network*, NeurIPS 2014 Deep Learning Workshop — [arXiv:1503.02531](https://arxiv.org/abs/1503.02531)
- Wang et al., *MiniLM: Deep Self-Attention Distillation for Task-Agnostic Compression of Pre-Trained Transformers*, NeurIPS 2020 — [arXiv:2002.10957](https://arxiv.org/abs/2002.10957)
- Askell et al., *A General Language Assistant as a Laboratory for Alignment*, 2021 — [arXiv:2112.00861](https://arxiv.org/abs/2112.00861). Context distillation is Section 2.1.
- Snell, Klein & Zhong, *Learning by Distilling Context*, 2022 — [arXiv:2209.15189](https://arxiv.org/abs/2209.15189)
- Huang et al., *In-context Learning Distillation: Transferring Few-shot Learning Ability of Pre-trained Language Models*, 2022 — [arXiv:2212.10670](https://arxiv.org/abs/2212.10670)
- von Oswald et al., *Transformers Learn In-Context by Gradient Descent*, ICML 2023 — [arXiv:2212.07677](https://arxiv.org/abs/2212.07677)
- Dai et al., *Why Can GPT Learn In-Context? Language Models Implicitly Perform Gradient Descent as Meta-Optimizers*, Findings of ACL 2023 — [arXiv:2212.10559](https://arxiv.org/abs/2212.10559)
- Agarwal et al., *On-Policy Distillation of Language Models: Learning from Self-Generated Mistakes* (GKD), ICLR 2024 — [arXiv:2306.13649](https://arxiv.org/abs/2306.13649)
- Lu, *On-Policy Distillation*, Thinking Machines Lab blog, October 2025 — [thinkingmachines.ai](https://thinkingmachines.ai/blog/on-policy-distillation/)
- Shenfeld, Pari & Agrawal, *RL's Razor: Why Online Reinforcement Learning Forgets Less*, ICLR 2026 — [arXiv:2509.04259](https://arxiv.org/abs/2509.04259)
- Shenfeld, Damani, Hübotter & Agrawal, *Self-Distillation Enables Continual Learning*, 2026 — [arXiv:2601.19897](https://arxiv.org/abs/2601.19897). Numbers above are from Table 5 of the January version.
- Ye et al., *On-Policy Context Distillation for Language Models*, 2026 — [arXiv:2602.12275](https://arxiv.org/abs/2602.12275)
- Yang, Kautz & Hatamizadeh, *Gated Delta Networks: Improving Mamba2 with Delta Rule*, ICLR 2025 — [arXiv:2412.06464](https://arxiv.org/abs/2412.06464)
- Sun et al., *Learning to (Learn at Test Time): RNNs with Expressive Hidden States*, 2024 — [arXiv:2407.04620](https://arxiv.org/abs/2407.04620)

The memory-trajectory figure is a sketch of the argument, not a measurement.

---

{% include citation.html
  title="Distillation in LLMs: From Context to Weights"
  author="Renee Jia"
  journal="renee-jia.github.io"
  year="2026"
  url="https://renee-jia.github.io/research%20blog/ai%20research/distillation-in-llms-from-context-to-weights/"
  bibtex_key="reneejia2026distillationllms"
%}
