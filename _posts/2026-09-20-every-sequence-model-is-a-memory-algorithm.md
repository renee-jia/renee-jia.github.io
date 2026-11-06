---
title: "Every Sequence Model Is a Memory Algorithm"
date: 2026-09-20
research_kind: open-problem
categories:
  - Research Blog
  - AI Research
tags:
  - sequence models
  - linear attention
  - DeltaNet
  - state space models
  - test-time training
  - associative memory
  - continual learning
excerpt: "A Transformer never changes its KV cache after writing to it. So how does it learn anything in context? Because every sequence model is really an online learning algorithm for its own memory. What separates them isn't FLOPs or context length. It's how fast they learn new things and how much of the old they keep."
read_time: "15-18 min read"
layout: distill
toc: false
last_modified_at: 2026-09-20T08:00:00-08:00
---

*Storage, write rules, forgetting rules, and the tradeoff none of them escapes.*

---

There's a small paradox at the heart of in-context learning.

A Transformer's KV cache never learns anything. It's a list. Keys and values go in, sit there untouched, and get dropped when the context ends. Nothing in that list adapts. The only thing that adapts is the read: a query comes in, softmax picks what to pull out. And yet we say Transformers learn in context, and empirically they do. Show one a few examples and it starts doing the task.

So where is the learning?

The answer is that memory and learning aren't separate things here. Any model that eats a stream of tokens is running an online algorithm. It decides what to keep, how to fold in what just arrived, and what to drop. The KV cache is the laziest possible version of that algorithm. Write rule: append. Forgetting rule: never. All the intelligence got pushed downstream into retrieval.

That reframing does a lot of work. The last few years gave us linear attention, DeltaNet, Mamba, RWKV, RetNet, test-time training, Titans, and a pile of systems built around retrieval. The papers mostly argue about perplexity and throughput. Read them as memory algorithms instead and the zoo shrinks. What's left is a design space with three axes, and a question continual learning has been stuck on for forty years: **how plastic can a memory be before it stops retaining anything?**

---

## Every memory mechanism answers three questions

Three questions pin down any memory in a sequence model. The answers turn out to be close to independent.

**Where does it live?** As a raw list of key–value pairs. In a fixed-size matrix. In a fixed-size state vector. In the weights of a small model that keeps training at inference time. Or outside the network, in an index.

**How is it written?** By appending. By stacking outer products on top of each other. With a gate that decides how much of the new input gets in. With an error-correcting update that checks what the memory already predicts for this key. By a gradient step. Or selectively, where something like surprise decides whether to write at all.

**How is it forgotten?** Never. By decay, where old content shrinks a little every step. By overwrite, where a new value pushes out the old one at the same key. By interference, where a fixed-size store blurs things together whether you want it to or not. By compression. Or by never being retrieved again.

<figure class="post-figure">
<svg viewBox="0 0 660 300" xmlns="http://www.w3.org/2000/svg" role="img" aria-label="Three independent axes: where memory lives, how it is written, how it is forgotten, with DeltaNet and Mamba traced as two paths">
<defs>
<marker id="m1a" markerWidth="7" markerHeight="7" refX="6" refY="3" orient="auto"><path d="M0,0 L6,3 L0,6 z" fill="#9a7048"/></marker>
<marker id="m1b" markerWidth="7" markerHeight="7" refX="6" refY="3" orient="auto"><path d="M0,0 L6,3 L0,6 z" fill="#6b7a52"/></marker>
</defs>
<style>.lb{font-family:'Source Sans 3',system-ui,sans-serif;font-size:10.5px;fill:#8a7b6b}.lbs{font-family:'Source Sans 3',system-ui,sans-serif;font-size:11px;fill:#3d342b}.hd{font-family:'Source Sans 3',system-ui,sans-serif;font-size:11.5px;fill:#3d342b;font-weight:600}</style>
<text x="20" y="30" class="hd">Where it lives</text>
<text x="245" y="30" class="hd">How it's written</text>
<text x="470" y="30" class="hd">How it's forgotten</text>

<rect x="20" y="52" width="176" height="24" rx="3" fill="#f7f1e7" stroke="#e4d5c4"/><text x="30" y="68" class="lbs">raw list of (k, v)</text>
<rect x="20" y="82" width="176" height="24" rx="3" fill="#f0e6d6" stroke="#9a7048"/><text x="30" y="98" class="lbs">d &#215; d matrix</text>
<rect x="20" y="112" width="176" height="24" rx="3" fill="#e6e9dc" stroke="#6b7a52"/><text x="30" y="128" class="lbs">state vector h</text>
<rect x="20" y="142" width="176" height="24" rx="3" fill="#f7f1e7" stroke="#e4d5c4"/><text x="30" y="158" class="lbs">model parameters &#952;</text>
<rect x="20" y="172" width="176" height="24" rx="3" fill="#f7f1e7" stroke="#e4d5c4"/><text x="30" y="188" class="lbs">external index</text>

<rect x="245" y="52" width="176" height="24" rx="3" fill="#f7f1e7" stroke="#e4d5c4"/><text x="255" y="68" class="lbs">append</text>
<rect x="245" y="82" width="176" height="24" rx="3" fill="#f7f1e7" stroke="#e4d5c4"/><text x="255" y="98" class="lbs">additive outer product</text>
<rect x="245" y="112" width="176" height="24" rx="3" fill="#e6e9dc" stroke="#6b7a52"/><text x="255" y="128" class="lbs">gated</text>
<rect x="245" y="142" width="176" height="24" rx="3" fill="#f0e6d6" stroke="#9a7048"/><text x="255" y="158" class="lbs">error-correcting (delta)</text>
<rect x="245" y="172" width="176" height="24" rx="3" fill="#f7f1e7" stroke="#e4d5c4"/><text x="255" y="188" class="lbs">gradient step</text>
<rect x="245" y="202" width="176" height="24" rx="3" fill="#f7f1e7" stroke="#e4d5c4"/><text x="255" y="218" class="lbs">surprise-gated</text>

<rect x="470" y="52" width="176" height="24" rx="3" fill="#f7f1e7" stroke="#e4d5c4"/><text x="480" y="68" class="lbs">never</text>
<rect x="470" y="82" width="176" height="24" rx="3" fill="#e6e9dc" stroke="#6b7a52"/><text x="480" y="98" class="lbs">decay</text>
<rect x="470" y="112" width="176" height="24" rx="3" fill="#f0e6d6" stroke="#9a7048"/><text x="480" y="128" class="lbs">overwrite</text>
<rect x="470" y="142" width="176" height="24" rx="3" fill="#f7f1e7" stroke="#e4d5c4"/><text x="480" y="158" class="lbs">interference</text>
<rect x="470" y="172" width="176" height="24" rx="3" fill="#f7f1e7" stroke="#e4d5c4"/><text x="480" y="188" class="lbs">compression</text>
<rect x="470" y="202" width="176" height="24" rx="3" fill="#f7f1e7" stroke="#e4d5c4"/><text x="480" y="218" class="lbs">retrieval policy</text>

<path d="M 196 94 C 218 94, 223 154, 245 154" stroke="#9a7048" stroke-width="1.3" fill="none" marker-end="url(#m1a)"/>
<path d="M 421 154 C 443 154, 448 124, 470 124" stroke="#9a7048" stroke-width="1.3" fill="none" marker-end="url(#m1a)"/>
<path d="M 196 124 H 243" stroke="#6b7a52" stroke-width="1.3" fill="none" marker-end="url(#m1b)"/>
<path d="M 421 124 C 443 124, 448 94, 470 94" stroke="#6b7a52" stroke-width="1.3" fill="none" marker-end="url(#m1b)"/>

<line x1="20" y1="252" x2="44" y2="252" stroke="#9a7048" stroke-width="1.3"/><text x="52" y="256" class="lb">DeltaNet &#8212; matrix, delta write, overwrite</text>
<line x1="20" y1="272" x2="44" y2="272" stroke="#6b7a52" stroke-width="1.3"/><text x="52" y="276" class="lb">Mamba &#8212; state vector, gated write, decay</text>
<text x="646" y="276" class="lb" text-anchor="end">most &#8220;new architectures&#8221; are a new path, not a new column</text>
</svg>
<figcaption>Three choices, close to independent. A gradient-descent write rule works on a hidden state (TTT) or on an external store. A decay forgetting rule works on an associative matrix (RetNet) or on a state vector (Mamba).</figcaption>
</figure>

Those columns being independent is what makes the grid worth drawing. Most of the novelty of the last three years is a new path through it, not a new column. That's not a criticism. Some of those paths work much better than what came before. But it means the comparisons worth making run along the axes, not between brand names.

The families below are sorted by how much the memory is allowed to learn. Start with the one where it learns nothing.

---

## Attention keeps everything, and pays for it

A Transformer's memory after $t$ tokens is

$$M_t = \{(k_1, v_1), \dots, (k_t, v_t)\},$$

and reading it is soft nearest-neighbor lookup. Raw storage, append, no forgetting. If something made it into the context, it's still in there. The only question is whether attention can find it.

You pay for that in the obvious way. Memory grows with the context, and every read scans all of it. Sliding-window attention, sparse attention, landmark tokens, Transformer-XL, the Compressive Transformer: each one changes the storage or the forgetting rule and leaves the read alone. Sliding windows add a hard cutoff. Drop anything older than $W$. Landmark attention keeps the whole history but changes the index. Each block gets a tag token, so the model finds the block first and attends inside it after. The Compressive Transformer swaps forgetting for compression, squeezing old chunks into summary vectors that become memory slots in their own right. In every case the read stays attention. What changes is what there is to attend over.

Reach for this family when you care more about keeping information than about the bill. It's also the family where memory and learning stay most cleanly apart. Nothing in the memory adapts. Only the read does.

Which leads to the question the rest of the field has been answering. What happens when the memory is too small to keep everything?

---

## A matrix blurs; the delta rule corrects

Collapse the list into a fixed-size matrix. Linear attention swaps the softmax kernel for a feature map and accumulates

$$M_t = M_{t-1} + v_t k_t^\top, \qquad y_t = M_t q_t.$$

Storage is now a $d \times d$ associative memory. The write is additive. Nothing is ever erased, so forgetting happens by accident, as blur. Every outer product lands on top of the last one. Query the memory and you get back a weighted sum of every value whose key looks like your query. Similar keys blur. Write the same key twice with different values and you get both back, added together. Nothing in that update can say *this key means something else now*.

The delta rule fixes it. Make the write depend on what's already in there:

$$M_t = M_{t-1} + \beta_t \big(v_t - M_{t-1} k_t\big) k_t^\top.$$

The term in parentheses is a prediction error. What the memory returns for $k_t$ today, minus what it should return. Schlag, Irie and Schmidhuber introduced this update when they showed linear Transformers are secretly fast weight programmers. Yang and colleagues later made it practical by running it in parallel across the sequence, which is where the name DeltaNet comes from. Forgetting is now overwrite. Write a key again and the old association gets corrected instead of buried. $\beta_t$ sets how hard.

<figure class="post-figure">
<svg viewBox="0 0 660 285" xmlns="http://www.w3.org/2000/svg" role="img" aria-label="The same key written twice: linear attention returns a superposition, DeltaNet returns the corrected value">
<defs>
<marker id="m2a" markerWidth="7" markerHeight="7" refX="6" refY="3" orient="auto"><path d="M0,0 L6,3 L0,6 z" fill="#b5a594"/></marker>
</defs>
<style>.lb{font-family:'Source Sans 3',system-ui,sans-serif;font-size:10.5px;fill:#8a7b6b}.lbs{font-family:'Source Sans 3',system-ui,sans-serif;font-size:11.5px;fill:#3d342b}.hd{font-family:'Source Sans 3',system-ui,sans-serif;font-size:11.5px;fill:#3d342b;font-weight:600}.mono{font-family:ui-monospace,monospace;font-size:11.5px;fill:#3d342b}</style>
<text x="18" y="22" class="hd">The same key, written twice.</text>
<text x="215" y="46" class="lb" text-anchor="middle">write (k, v&#8321;)</text>
<text x="395" y="46" class="lb" text-anchor="middle">later: write (k, v&#8322;)</text>
<text x="576" y="46" class="lb" text-anchor="middle">read with q = k</text>

<text x="18" y="80" class="hd">Linear attention</text>
<text x="18" y="96" class="lb">additive, Hebbian</text>
<rect x="150" y="58" width="130" height="52" rx="3" fill="#f7f1e7" stroke="#e4d5c4"/>
<text x="215" y="90" class="mono" text-anchor="middle">M = v&#8321;k<tspan font-size="8" dy="-4">T</tspan></text>
<path d="M 284 84 H 306" stroke="#b5a594" stroke-width="1.2" marker-end="url(#m2a)"/>
<rect x="310" y="58" width="170" height="52" rx="3" fill="#f7f1e7" stroke="#e4d5c4"/>
<text x="395" y="90" class="mono" text-anchor="middle">M = v&#8321;k<tspan font-size="8" dy="-4">T</tspan><tspan dy="4"> + v&#8322;k</tspan><tspan font-size="8" dy="-4">T</tspan></text>
<path d="M 484 84 H 506" stroke="#b5a594" stroke-width="1.2" marker-end="url(#m2a)"/>
<rect x="510" y="58" width="132" height="52" rx="3" fill="#f0dcd4" stroke="#a8543f"/>
<text x="576" y="90" class="mono" text-anchor="middle" fill="#a8543f">Mk = v&#8321; + v&#8322;</text>
<text x="150" y="132" class="lb" fill="#a8543f">the old value is still in there, and the read returns both</text>

<line x1="18" y1="152" x2="642" y2="152" stroke="#e4d5c4" stroke-width="1"/>

<text x="18" y="200" class="hd">DeltaNet</text>
<text x="18" y="216" class="lb">error-correcting</text>
<rect x="150" y="178" width="130" height="52" rx="3" fill="#f7f1e7" stroke="#e4d5c4"/>
<text x="215" y="210" class="mono" text-anchor="middle">M = v&#8321;k<tspan font-size="8" dy="-4">T</tspan></text>
<path d="M 284 204 H 306" stroke="#b5a594" stroke-width="1.2" marker-end="url(#m2a)"/>
<rect x="310" y="178" width="170" height="52" rx="3" fill="#f0e6d6" stroke="#9a7048"/>
<text x="395" y="210" class="mono" text-anchor="middle">M = v&#8321;k<tspan font-size="8" dy="-4">T</tspan><tspan dy="4"> + (v&#8322;&#8722;v&#8321;)k</tspan><tspan font-size="8" dy="-4">T</tspan></text>
<path d="M 484 204 H 506" stroke="#b5a594" stroke-width="1.2" marker-end="url(#m2a)"/>
<rect x="510" y="178" width="132" height="52" rx="3" fill="#e6e9dc" stroke="#6b7a52"/>
<text x="576" y="210" class="mono" text-anchor="middle" fill="#6b7a52">Mk = v&#8322;</text>
<text x="150" y="252" class="lb" fill="#6b7a52">the write is the prediction error, so the old association is corrected</text>
<text x="150" y="270" class="lb">(both lines assume &#946; = 1 and unit-norm k; the difference is the update rule, not the tuning)</text>
</svg>
<figcaption>Linear attention and DeltaNet differ by one term. That term is the difference between remembering a key's history and knowing its current value.</figcaption>
</figure>

This is where the framing starts paying for itself. $M$ *is* a linear model. The write rule *is* its training algorithm: one step of online gradient descent on squared error. In-context learning here isn't like online learning. It is online learning, running in the hidden state.

Linear attention does Hebbian learning. DeltaNet does error-corrected learning. So when the two behave differently on recall tasks with reused keys, that isn't an architecture mystery. It's the gap between two learning rules that people understood decades before either paper.

That's one way to shrink a memory. There's a second, and it starts from a different picture of what a state is.

---

## A state vector forgets by compressing

The other family keeps a fixed-size state too. It just doesn't treat it as a key–value table. It treats it as a compressed summary of everything so far:

$$h_t = A_t h_{t-1} + B_t x_t.$$

S4, Mamba, Mamba-2 and the selective SSMs live here. So do RWKV, HGRN, RetNet and gated linear attention. What they share is that the old state gets multiplied by something before the new input is added. RetNet uses a fixed decay,

$$S_t = \gamma S_{t-1} + k_t v_t^\top,$$

which is linear attention with a forgetting rule added on. Mamba makes the transition input-dependent, so the model picks per token how much of the past survives. Gated DeltaNet does the obvious thing once you see write and forgetting as separate knobs: delta write, gated forget. RWKV-7 lands somewhere similar from the other side, with a generalized delta rule driving its state.

The research question here is a different one. Nobody asks whether Mamba can overwrite an association. They ask how much history a $d$-dimensional state can hold, and what the gate learns to throw away. The failure mode isn't blur between similar keys. It's information that never made it into the state at all. Arora and colleagues pinned this down in the Zoology work. Recall quality tracks recurrent state size, and you don't get recall and throughput for free at the same time. Jelassi and colleagues came at it from the other end. They showed Transformers beat SSMs at plain copying, about as pure a test of "did you keep it" as you can build.

Keep the two pictures separate even when the equations line up. DeltaNet is *memory as an online learner*. Mamba is *memory as a dynamical system*. They suggest different checks, different ablations, different stories about why a run went wrong. Conversations go wrong when two people use the same equation to mean different things.

---

## If the memory is a learner, give it a better learner

Push the learner reading one step further. DeltaNet's memory is a linear model trained by one gradient step per token. Why linear?

Test-time training says it doesn't have to be. The memory becomes a small network $f_\theta$, and every token triggers an update:

$$\theta_t = \theta_{t-1} - \eta \nabla_\theta \mathcal{L}(x_t; \theta), \qquad y_t = f_{\theta_t}(q_t).$$

Make $f$ linear and $\mathcal{L}$ squared error, and DeltaNet drops out. Make it an MLP and the memory can hold nonlinear structure. Wang and colleagues turned this into a recipe: pick an associative-memory objective, pick an optimizer, read off an architecture. That's roughly where the field has landed, even if the papers still arrive one architecture at a time.

Titans push somewhere else. Not on how expressive the memory is, but on *what gets written*. The long-term memory still updates by gradient descent. But the size of each write is set by surprise: how far off the memory's prediction for the current token was. Tokens the memory already predicts barely move it. Novel tokens write hard. Add momentum so surprise carries for a few steps. Add weight decay as a forgetting gate. Now the memory can tell "this is new, keep it" from "this is routine, skip it."

<figure class="post-figure">
<svg viewBox="0 0 660 300" xmlns="http://www.w3.org/2000/svg" role="img" aria-label="Two directions to improve a memory-learner: a better learner along the horizontal axis, more selective writes along the vertical">
<defs>
<marker id="m3a" markerWidth="7" markerHeight="7" refX="6" refY="3" orient="auto"><path d="M0,0 L6,3 L0,6 z" fill="#b5a594"/></marker>
<marker id="m3b" markerWidth="7" markerHeight="7" refX="6" refY="3" orient="auto"><path d="M0,0 L6,3 L0,6 z" fill="#9a7048"/></marker>
</defs>
<style>.lb{font-family:'Source Sans 3',system-ui,sans-serif;font-size:10px;fill:#8a7b6b}.lbs{font-family:'Source Sans 3',system-ui,sans-serif;font-size:11.5px;fill:#3d342b}.hd{font-family:'Source Sans 3',system-ui,sans-serif;font-size:11.5px;fill:#3d342b;font-weight:600}.mono{font-family:ui-monospace,monospace;font-size:10px;fill:#8a7b6b}</style>
<text x="52" y="24" class="hd">Two directions to push a memory-learner.</text>

<line x1="40" y1="266" x2="40" y2="92" stroke="#b5a594" stroke-width="1" marker-end="url(#m3a)"/>
<text x="26" y="180" class="lb" text-anchor="middle" transform="rotate(-90 26 180)">more selective writes</text>
<line x1="52" y1="270" x2="608" y2="270" stroke="#b5a594" stroke-width="1" marker-end="url(#m3a)"/>
<text x="330" y="290" class="lb" text-anchor="middle">better memory-learner</text>

<rect x="52" y="184" width="160" height="56" rx="3" fill="#f7f1e7" stroke="#e4d5c4"/>
<text x="132" y="208" class="lbs" text-anchor="middle">Linear attention</text>
<text x="132" y="226" class="mono" text-anchor="middle">M &#8592; M + vk<tspan font-size="7.5" dy="-3">T</tspan></text>

<path d="M 214 212 H 242" stroke="#b5a594" stroke-width="1.2" marker-end="url(#m3a)"/>

<rect x="246" y="184" width="160" height="56" rx="3" fill="#f7f1e7" stroke="#e4d5c4"/>
<text x="326" y="208" class="lbs" text-anchor="middle">DeltaNet</text>
<text x="326" y="226" class="mono" text-anchor="middle">+ &#946;(v &#8722; Mk)k<tspan font-size="7.5" dy="-3">T</tspan></text>

<path d="M 408 212 H 436" stroke="#b5a594" stroke-width="1.2" marker-end="url(#m3a)"/>

<rect x="440" y="184" width="160" height="56" rx="3" fill="#f0e6d6" stroke="#9a7048"/>
<text x="520" y="208" class="lbs" text-anchor="middle">Test-time training</text>
<text x="520" y="226" class="mono" text-anchor="middle">&#952; &#8592; &#952; &#8722; &#951;&#8711;L(x)</text>

<path d="M 520 182 V 152" stroke="#9a7048" stroke-width="1.2" marker-end="url(#m3b)"/>

<rect x="440" y="94" width="160" height="56" rx="3" fill="#f0e6d6" stroke="#9a7048"/>
<text x="520" y="118" class="lbs" text-anchor="middle">Titans</text>
<text x="520" y="136" class="mono" text-anchor="middle">write &#8733; surprise</text>

<text x="52" y="60" class="lb">Horizontal: how good a learner the memory is. Vertical: which tokens earn a write at all.</text>
<text x="52" y="76" class="lb">Nothing forces a model to move along only one of them, and few models move along both.</text>
</svg>
<figcaption>Learner quality and selectivity are separate knobs. Most of the last three years of work has turned the horizontal one.</figcaption>
</figure>

That second axis is the interesting one. Uniform updates versus importance-weighted updates. It's the least explored knob in the space. It's also where continual learning has the most to say, and gets listened to the least. Every architecture above writes on every token. Biological memory doesn't. Neither does anyone who takes decent notes.

It also needs numbers and doesn't have many. The obvious experiment: a stream whose key distribution shifts halfway through, a linear-attention memory and a TTT memory at matched state size. Don't score final accuracy. Score two curves. How fast does each pick up the new pattern, and how much of the old one can it still answer? The shapes should differ more than the endpoints do. That's a guess, not a result.

---

## Outside the network, forgetting becomes a retrieval problem

One family refuses the question. Why compress the past into the network at all?

kNN-LM, the Memorizing Transformer, RETRO, RAG, most agent memory systems: they write events to a store outside the model,

$$m_i = (k_i, v_i), \qquad i^* = \arg\max_i \; \mathrm{sim}(q, k_i),$$

and look them up at read time. There's no real limit on capacity. Nothing gets overwritten by accident. RETRO took it to the limit and retrieved from a database of trillions of tokens. The Memorizing Transformer showed you can bolt a kNN lookup onto an attention layer and let the model learn to use it.

The hard problems don't go away. They move to the edge of the store. What's worth writing? How do you index it so the right thing comes back? Should the model retrieve at all, or trust its weights? What happens when a memory is out of date, or when two memories disagree? Over long horizons those questions matter more than capacity. A store with a bad write policy is a growing pile of noise. A store with a bad read policy is a library with no index.

<figure class="post-figure">
<svg viewBox="0 0 660 262" xmlns="http://www.w3.org/2000/svg" role="img" aria-label="A funnel from all experience to what is actually used, with losses at each stage">
<defs><marker id="m4a" markerWidth="7" markerHeight="7" refX="6" refY="3" orient="auto"><path d="M0,0 L6,3 L0,6 z" fill="#b5a594"/></marker></defs>
<style>.lb{font-family:'Source Sans 3',system-ui,sans-serif;font-size:10px;fill:#8a7b6b}.lbs{font-family:'Source Sans 3',system-ui,sans-serif;font-size:11px;fill:#3d342b}.hd{font-family:'Source Sans 3',system-ui,sans-serif;font-size:11.5px;fill:#3d342b;font-weight:600}</style>
<text x="18" y="24" class="hd">Nothing is overwritten. Plenty is still lost.</text>

<text x="100" y="58" class="lbs" text-anchor="middle">everything that happened</text>
<rect x="40" y="70" width="120" height="140" rx="3" fill="#f3ebe0" stroke="#e4d5c4"/>

<path d="M 166 140 H 194" stroke="#b5a594" stroke-width="1.2" marker-end="url(#m4a)"/>

<text x="260" y="58" class="lbs" text-anchor="middle">written to the store</text>
<rect x="200" y="84" width="120" height="112" rx="3" fill="#f3ebe0" stroke="#e4d5c4"/>

<path d="M 326 140 H 354" stroke="#b5a594" stroke-width="1.2" marker-end="url(#m4a)"/>

<text x="420" y="58" class="lbs" text-anchor="middle">surfaced by the index</text>
<rect x="360" y="107" width="120" height="66" rx="3" fill="#ecdfcd" stroke="#e4d5c4"/>

<path d="M 486 140 H 514" stroke="#b5a594" stroke-width="1.2" marker-end="url(#m4a)"/>

<text x="575" y="58" class="lbs" text-anchor="middle">actually used</text>
<rect x="520" y="123" width="110" height="34" rx="3" fill="#f0e6d6" stroke="#9a7048"/>

<text x="180" y="232" class="lb" text-anchor="middle" fill="#9a7048">what was worth</text>
<text x="180" y="245" class="lb" text-anchor="middle" fill="#9a7048">writing down?</text>
<text x="340" y="232" class="lb" text-anchor="middle" fill="#9a7048">does the query</text>
<text x="340" y="245" class="lb" text-anchor="middle" fill="#9a7048">match the key?</text>
<text x="502" y="232" class="lb" text-anchor="middle" fill="#9a7048">top-k crowding,</text>
<text x="502" y="245" class="lb" text-anchor="middle" fill="#9a7048">staleness, contradiction</text>
</svg>
<figcaption>External memory moves forgetting out of the substrate and into the retrieval policy. The past is never lost; it is just not found.</figcaption>
</figure>

In the three-axis language: unlimited storage, a write rule nobody has specified, and a forgetting rule handed to a similarity function. That's a strange place to park, and it won't hold.

---

## It's the stability–plasticity dilemma in an architecture costume

Line the families up and the same shape shows up every time. Each one is good on one side of a single tradeoff and pays for it on the other. One word needs pinning down first, because *plasticity* gets used two ways. Here it means how much one new token is allowed to change what the memory already holds. *Retention* means how much of what was stored can still be read back. Both are properties of the memory, not of the model around it.

That's what separates the first two rows. Neither ever erases anything. But KV attention keeps every pair separate, and linear attention sums them into one matrix, so its reads blur. Mamba and RWKV often get filed under linear attention too. Here they're the gated row, because the decay is what moves them along the curve.

| Substrate | Plasticity | Retention | Forgetting curve | Failure mode |
|---|---|---|---|---|
| KV attention | None in the store. A token adds a pair and changes nothing; the read does the adapting | Perfect within the window, zero past it | Flat, then a cliff | Context is finite; nothing survives past it |
| Linear attention | Low. Additive, so a reused key is out-voted, not replaced | High but blurred; every read mixes values together | Not a function of time; blur grows with the number of distinct keys | Interference |
| DeltaNet | High. A reused key is overwritten | Moderate | Not a function of time; drops at each reassignment | Overwrite destroys the old value |
| SSM / gated RNN (Mamba, RWKV, RetNet) | Gate-controlled | Gate-controlled; below KV attention at every lag within the window | Smooth decay. Exponential for a fixed rate, closer to a power law when rates vary by channel | Compression loss; a wrong gate can't be undone |
| Test-time training | Very high | Fragile | Steep | Catastrophic forgetting in the fast weights |
| Titans-style | Selective | Improved | Decay, except for surprising tokens | Only as good as the surprise signal |
| External memory | None in the store; it only grows | Strong, if retrieved | Flat; loss happens at retrieval, not over time | Retrieval is the bottleneck; out-of-date or conflicting entries |

Two rows don't play the game. KV attention and the external store never change what they hold. They grow instead. That's not a resolution of the tradeoff. It's a way of buying out of it, one slot per token. The bill comes due at the window, or at the retriever. Every fixed-size memory has to pay on one side or the other.

That tradeoff has a name. Grossberg called it the stability–plasticity dilemma back in 1980, and continual learning still hasn't found a way around it. Put the two sides on axes,

$$\begin{aligned}
\text{Plasticity} &= \text{how much one new token can change what is already stored}, \\
\text{Retention} &= \text{how much of what was stored can still be read back},
\end{aligned}$$

and every architecture above is a point on that plane. Change its hyperparameters and it traces a curve.

<figure class="post-figure">
<svg viewBox="0 0 660 340" xmlns="http://www.w3.org/2000/svg" role="img" aria-label="Architectures plotted on a plasticity versus retention plane. KV attention sits alone in the top-left corner, with the external store beside it; both are drawn as hollow circles because their stores grow rather than change. Linear attention, the gated SSM, DeltaNet and test-time training sit on a dashed frontier running down and to the right. Titans-style sits directly above test-time training.">
<defs><marker id="m5a" markerWidth="7" markerHeight="7" refX="6" refY="3" orient="auto"><path d="M0,0 L6,3 L0,6 z" fill="#b5a594"/></marker></defs>
<style>.lb{font-family:'Source Sans 3',system-ui,sans-serif;font-size:10.5px;fill:#3d342b}.lbm{font-family:'Source Sans 3',system-ui,sans-serif;font-size:10px;fill:#8a7b6b}.hd{font-family:'Source Sans 3',system-ui,sans-serif;font-size:11.5px;fill:#3d342b;font-weight:600}</style>
<text x="90" y="28" class="hd">Every architecture is a point on the same plane.</text>

<line x1="90" y1="290" x2="90" y2="48" stroke="#b5a594" stroke-width="1" marker-end="url(#m5a)"/>
<line x1="90" y1="290" x2="622" y2="290" stroke="#b5a594" stroke-width="1" marker-end="url(#m5a)"/>
<text x="56" y="175" class="lbm" text-anchor="middle" transform="rotate(-90 56 175)">retention &#8594;</text>
<text x="356" y="322" class="lbm" text-anchor="middle">plasticity: how much one token can change the store &#8594;</text>

<path d="M 232 140 C 345 160, 450 200, 555 270" stroke="#b5a594" stroke-width="1.1" fill="none" stroke-dasharray="4 3"/>

<text x="128" y="52" class="lbm">keeps more, changes less</text>
<circle cx="118" cy="68" r="4.5" fill="#fff" stroke="#9a7048" stroke-width="1.6"/><text x="128" y="72" class="lb">KV attention, within the window</text>
<text x="128" y="86" class="lbm">nothing in the store ever changes</text>
<line x1="118" y1="77" x2="118" y2="284" stroke="#a8543f" stroke-width="1" stroke-dasharray="3 3"/>
<text x="118" y="306" class="lbm" text-anchor="middle" fill="#a8543f">window ends</text>

<circle cx="150" cy="106" r="4.5" fill="#fff" stroke="#9a7048" stroke-width="1.6"/><text x="160" y="110" class="lb">external memory, if retrieval finds it</text>

<circle cx="255" cy="146" r="4" fill="#9a7048"/><text x="265" y="150" class="lb">linear attention</text>

<line x1="493" y1="193" x2="493" y2="227" stroke="#b5a594" stroke-width="1" stroke-dasharray="2 2"/>
<circle cx="493" cy="188" r="4" fill="#9a7048"/><text x="503" y="192" class="lb">Titans-style</text>
<circle cx="372" cy="181" r="4" fill="#9a7048"/><text x="382" y="185" class="lb">SSM / gated RNN</text>
<circle cx="445" cy="207" r="4" fill="#9a7048"/><text x="455" y="211" class="lb">DeltaNet</text>
<circle cx="493" cy="232" r="4" fill="#9a7048"/><text x="503" y="236" class="lb">TTT + weight decay</text>
<circle cx="555" cy="270" r="4" fill="#9a7048"/><text x="545" y="274" class="lb" text-anchor="end">TTT, large &#951;</text>
<text x="618" y="306" class="lbm" text-anchor="end">changes more, keeps less</text>

<circle cx="152" cy="236" r="4.5" fill="#fff" stroke="#9a7048" stroke-width="1.6"/><text x="166" y="240" class="lbm">the store grows; nothing already in it ever changes</text>
<circle cx="152" cy="256" r="4" fill="#9a7048"/><text x="166" y="260" class="lbm">fixed size; a write has to displace something</text>
<line x1="140" y1="276" x2="164" y2="276" stroke="#b5a594" stroke-width="1.1" stroke-dasharray="4 3"/><text x="166" y="280" class="lbm">the frontier for any fixed-size memory</text>
</svg>
<figcaption>Positions are illustrative, not measured. KV attention owns the corner: its store never changes, so retention is perfect until the window ends. Linear attention never erases either, but its reads blur, so it starts the frontier instead. Decay and overwriting slide a point down the curve. Titans sits above TTT: same write rule, but only surprising tokens get to use it.</figcaption>
</figure>

The retention axis hides a shape. Plot how much of a token the memory can still return against how long ago it was written. The families separate again, this time by the form of the curve rather than its height.

<figure class="post-figure">
<svg viewBox="0 0 660 300" xmlns="http://www.w3.org/2000/svg" role="img" aria-label="Forgetting curves: retention against lag since a token was written. KV attention is flat then drops to zero at the window. A gated SSM starts lower and fades smoothly, continuing past the window. Test-time training with a large step fades fastest. A dotted curve shows the Ebbinghaus shape for comparison.">
<defs><marker id="m6a" markerWidth="7" markerHeight="7" refX="6" refY="3" orient="auto"><path d="M0,0 L6,3 L0,6 z" fill="#b5a594"/></marker></defs>
<style>.lb{font-family:'Source Sans 3',system-ui,sans-serif;font-size:10.5px;fill:#3d342b}.lbm{font-family:'Source Sans 3',system-ui,sans-serif;font-size:10px;fill:#8a7b6b}.hd{font-family:'Source Sans 3',system-ui,sans-serif;font-size:11.5px;fill:#3d342b;font-weight:600}</style>
<text x="70" y="26" class="hd">Same axis, different shapes.</text>

<line x1="70" y1="250" x2="70" y2="40" stroke="#b5a594" stroke-width="1" marker-end="url(#m6a)"/>
<line x1="70" y1="250" x2="634" y2="250" stroke="#b5a594" stroke-width="1" marker-end="url(#m6a)"/>
<text x="50" y="145" class="lbm" text-anchor="middle" transform="rotate(-90 50 145)">how much can be read back &#8594;</text>
<text x="352" y="288" class="lbm" text-anchor="middle">lag since the token was written &#8594;</text>

<line x1="400" y1="44" x2="400" y2="250" stroke="#a8543f" stroke-width="1" stroke-dasharray="3 3"/>
<text x="400" y="268" class="lbm" text-anchor="middle" fill="#a8543f">window ends</text>

<path d="M 70 62 H 400 V 248" stroke="#9a7048" stroke-width="1.8" fill="none"/>
<text x="78" y="55" class="lb" fill="#9a7048">KV attention: verbatim inside the window, nothing after it</text>

<path d="M 70 62 C 100 110, 150 140, 240 155 S 460 175, 630 184" stroke="#b5a594" stroke-width="1.2" fill="none" stroke-dasharray="1.5 3"/>
<line x1="412" y1="157" x2="426" y2="157" stroke="#b5a594" stroke-width="1.2" stroke-dasharray="1.5 3"/><text x="432" y="160" class="lbm">human recall, the Ebbinghaus shape</text>

<path d="M 70 90 C 110 140, 170 165, 250 178 S 440 200, 630 210" stroke="#6b7a52" stroke-width="1.8" fill="none"/>
<text x="628" y="228" class="lb" fill="#6b7a52" text-anchor="end">SSM / gated RNN: Mamba, RWKV, RetNet</text>

<path d="M 70 72 C 90 175, 120 222, 190 234 L 630 240" stroke="#a8543f" stroke-width="1.3" fill="none"/>
<text x="175" y="226" class="lbm" fill="#a8543f">TTT with a large step: fastest to fade</text>

<line x1="300" y1="66" x2="300" y2="184" stroke="#3d342b" stroke-width="0.8"/>
<line x1="296" y1="66" x2="304" y2="66" stroke="#3d342b" stroke-width="0.8"/>
<line x1="296" y1="184" x2="304" y2="184" stroke="#3d342b" stroke-width="0.8"/>
<text x="306" y="110" class="lbm">the absolute</text>
<text x="306" y="123" class="lbm">retention gap</text>

<text x="412" y="100" class="lbm">past the window, only the slopes are left</text>
</svg>
<figcaption>Heights are illustrative. The shapes are the argument. Inside the window, KV attention's curve is above the gated state's at every lag. Past it, the order flips. Linear attention and DeltaNet aren't drawn because their forgetting isn't a function of lag. The human curve is shown for its form only, not its level.</figcaption>
</figure>

KV attention's curve is a step. Inside the window, everything is there, verbatim, at lag one and at lag ten thousand. Past the window, nothing is. The gated row forgets on a slope instead. Every step multiplies the state by a number a little below one. So whatever a token wrote fades a bit with each token that follows. In RetNet that number is fixed, and the fade is a clean exponential. In Mamba each channel of the state has its own rate, and the input sets that rate token by token. Recent versions of RWKV do the same. Add up many exponentials with different rates and the sum no longer looks like an exponential. It looks like a power law: steep early loss, then a long tail. That's the shape Ebbinghaus drew in 1885 and the one Wixted and Ebbesen fit to human recall a century later. So the slope is the human-looking one, and the step isn't. Nobody keeps a conversation verbatim for eight thousand words and then loses all of it at once.

But look at where the curves sit, not only how they bend. Inside the window the gated state's curve is under KV attention's everywhere. A gated state is lossy from the moment a token is written, and it keeps losing. The copying results from Jelassi and colleagues and the recall results from the Zoology paper both measure this gap. Past the window the order flips, because KV attention has nothing left and the state still has its tail. Mamba and RWKV forget more like a person does, and they remember less than a Transformer does. Both things are true, and which one matters depends on whether your task fits inside the window.

DeltaNet isn't on that plot, and that's informative. Its forgetting isn't a function of lag at all. An association survives until the same key is written again, then it's gone, however old or new it was. Linear attention is missing for a related reason. What blurs it is how many different keys have been written since, not how much time has passed. Lag is only the right axis for the memories that decay.

Curves like these are a better comparison than another Mamba-versus-Transformer-versus-TTT benchmark, because you can control what produces them. Build an online task whose statistics shift partway through. Keys get reassigned. Popularity follows a heavy tail. A new key family shows up in a burst. Junk arrives that should be ignored. Then ask each substrate two things. How fast does it pick up the new pattern? How much of the old one does it still get right? The answers aren't scalars. They're curves, and the curve is the architecture's signature.

A few predictions fall out, and all of them are cheap to test:

- Linear attention will be slow to pick up a reassigned key. The new value has to out-vote the old one, and that takes repeated writes. It will also get blurrier on every key, old and new, as the number of *distinct* keys grows. What hurts it is interference, not forgetting.
- DeltaNet cares less about how many keys there are. What hurts it is how often a key changes meaning. Each reassignment wipes out the old value, so retention should drop as the *reassignment rate* goes up.
- A TTT memory with an MLP should adapt fastest when the stream shifts, and forget the most. Regularize it and the gap closes. The fixes that work should be the ones continual learning already knows: decay toward the starting weights, an EWC-style penalty, replay.
- Surprise-gated writes should help when most tokens are routine, and hurt when the surprise signal is off. The obvious trap is junk that's novel but useless. Real data is full of that.

---

## Maybe worth studying more

Going in, the differences between these architectures looked like efficiency differences. That's how the papers frame them. Coming out, efficiency looks more like the constraint and memory policy like the actual content. A few questions kept showing up no matter which family I read, and none of them have clean answers yet.

**How much can a fixed state hold?** For SSMs this is the whole question. We have real capacity results for the linear case and mostly intuition for the selective one.

**What deserves to be written?** Surprise is one signal. Reward, downstream utility and predicted future relevance are others. None of them is obviously right. It's the same question as what an agent should put in its scratchpad, and it's just as open there. [An earlier post on agent memory](/research%20blog/ai%20research/from-memory-to-learning-continual-learning-for-llm-agents/) ran into the same wall from the other side.

**Is forgetting a bug or a feature?** In classical continual learning it's the bug with a name. In a memory that has to survive a shifting stream, decay is doing real work. It encodes a prior: recent information matters more. Architectures that *can't* forget, plain linear attention and ever-growing stores among them, aren't obviously better than the ones that can. Over long horizons they're probably worse.

**Can you compose substrates?** The most interesting recent systems mix several kinds. Attention for sharp short-term recall. A recurrent or fast-weight state for compressed medium-term context. An external index for the long tail. That's roughly the human split between working, semantic and episodic memory, which is either encouraging or a sign of seeing patterns that aren't there. Either way the question stops being which substrate wins. It becomes how to coordinate the write and forgetting rules across several at once. Nobody has a good answer to that yet.

A sequence architecture is an online learning algorithm for its own memory. The axis that matters most isn't FLOPs or context length. It's where the design sits on the plasticity–retention frontier. Most papers report the first two. Almost none report the third.

---

## Notes

References:

- Katharopoulos et al., *Transformers are RNNs: Fast Autoregressive Transformers with Linear Attention*, ICML 2020 — [arXiv:2006.16236](https://arxiv.org/abs/2006.16236)
- Ba et al., *Using Fast Weights to Attend to the Recent Past*, NIPS 2016 — [arXiv:1610.06258](https://arxiv.org/abs/1610.06258)
- Schlag, Irie & Schmidhuber, *Linear Transformers Are Secretly Fast Weight Programmers*, ICML 2021 — [arXiv:2102.11174](https://arxiv.org/abs/2102.11174)
- Yang et al., *Parallelizing Linear Transformers with the Delta Rule over Sequence Length* (DeltaNet), NeurIPS 2024 — [arXiv:2406.06484](https://arxiv.org/abs/2406.06484)
- Yang, Kautz & Hatamizadeh, *Gated Delta Networks: Improving Mamba2 with Delta Rule*, ICLR 2025 — [arXiv:2412.06464](https://arxiv.org/abs/2412.06464)
- Yang et al., *Gated Linear Attention Transformers with Hardware-Efficient Training*, ICML 2024 — [arXiv:2312.06635](https://arxiv.org/abs/2312.06635)
- Gu, Goel & Ré, *Efficiently Modeling Long Sequences with Structured State Spaces* (S4), ICLR 2022 — [arXiv:2111.00396](https://arxiv.org/abs/2111.00396)
- Gu & Dao, *Mamba: Linear-Time Sequence Modeling with Selective State Spaces*, COLM 2024 — [arXiv:2312.00752](https://arxiv.org/abs/2312.00752)
- Dao & Gu, *Transformers are SSMs: Generalized Models and Efficient Algorithms Through Structured State Space Duality* (Mamba-2), ICML 2024 — [arXiv:2405.21060](https://arxiv.org/abs/2405.21060)
- Sun et al., *Retentive Network: A Successor to Transformer for Large Language Models*, 2023 — [arXiv:2307.08621](https://arxiv.org/abs/2307.08621)
- Peng et al., *RWKV: Reinventing RNNs for the Transformer Era*, Findings of EMNLP 2023 — [arXiv:2305.13048](https://arxiv.org/abs/2305.13048)
- Peng et al., *Eagle and Finch: RWKV with Matrix-Valued States and Dynamic Recurrence* (RWKV-5 and RWKV-6), 2024 — [arXiv:2404.05892](https://arxiv.org/abs/2404.05892)
- Peng et al., *RWKV-7 "Goose" with Expressive Dynamic State Evolution*, 2025 — [arXiv:2503.14456](https://arxiv.org/abs/2503.14456)
- Qin, Yang & Zhong, *Hierarchically Gated Recurrent Neural Network for Sequence Modeling* (HGRN), NeurIPS 2023 — [arXiv:2311.04823](https://arxiv.org/abs/2311.04823)
- Sun et al., *Learning to (Learn at Test Time): RNNs with Expressive Hidden States* (TTT), 2024 — [arXiv:2407.04620](https://arxiv.org/abs/2407.04620)
- Behrouz, Zhong & Mirrokni, *Titans: Learning to Memorize at Test Time*, NeurIPS 2025 — [arXiv:2501.00663](https://arxiv.org/abs/2501.00663)
- Behrouz et al., *It's All Connected: A Journey Through Test-Time Memorization, Attentional Bias, Retention, and Online Optimization*, 2025 — [arXiv:2504.13173](https://arxiv.org/abs/2504.13173)
- Behrouz et al., *ATLAS: Learning to Optimally Memorize the Context at Test Time*, 2025 — [arXiv:2505.23735](https://arxiv.org/abs/2505.23735)
- Wang, Nguyen & Duvenaud, *Test-Time Regression: A Unifying Framework for Designing Sequence Models with Associative Memory*, 2025 — [arXiv:2501.12352](https://arxiv.org/abs/2501.12352)
- Dai et al., *Transformer-XL: Attentive Language Models Beyond a Fixed-Length Context*, ACL 2019 — [arXiv:1901.02860](https://arxiv.org/abs/1901.02860)
- Rae et al., *Compressive Transformers for Long-Range Sequence Modelling*, ICLR 2020 — [arXiv:1911.05507](https://arxiv.org/abs/1911.05507)
- Mohtashami & Jaggi, *Landmark Attention: Random-Access Infinite Context Length for Transformers*, NeurIPS 2023 — [arXiv:2305.16300](https://arxiv.org/abs/2305.16300)
- Munkhdalai, Faruqui & Gopal, *Leave No Context Behind: Efficient Infinite Context Transformers with Infini-attention*, 2024 — [arXiv:2404.07143](https://arxiv.org/abs/2404.07143)
- Khandelwal et al., *Generalization through Memorization: Nearest Neighbor Language Models* (kNN-LM), ICLR 2020 — [arXiv:1911.00172](https://arxiv.org/abs/1911.00172)
- Wu et al., *Memorizing Transformers*, ICLR 2022 — [arXiv:2203.08913](https://arxiv.org/abs/2203.08913)
- Borgeaud et al., *Improving Language Models by Retrieving from Trillions of Tokens* (RETRO), ICML 2022 — [arXiv:2112.04426](https://arxiv.org/abs/2112.04426)
- Lewis et al., *Retrieval-Augmented Generation for Knowledge-Intensive NLP Tasks*, NeurIPS 2020 — [arXiv:2005.11401](https://arxiv.org/abs/2005.11401)
- Arora et al., *Zoology: Measuring and Improving Recall in Efficient Language Models*, ICLR 2024 — [arXiv:2312.04927](https://arxiv.org/abs/2312.04927)
- Arora et al., *Simple Linear Attention Language Models Balance the Recall-Throughput Tradeoff* (Based), ICML 2024 — [arXiv:2402.18668](https://arxiv.org/abs/2402.18668)
- Jelassi et al., *Repeat After Me: Transformers are Better than State Space Models at Copying*, ICML 2024 — [arXiv:2402.01032](https://arxiv.org/abs/2402.01032)
- Kirkpatrick et al., *Overcoming Catastrophic Forgetting in Neural Networks* (EWC), PNAS 2017 — [arXiv:1612.00796](https://arxiv.org/abs/1612.00796)

The stability–plasticity dilemma predates all of this; the standard reference is Grossberg's adaptive resonance work from the 1970s and 1980s, later summarized in Carpenter and Grossberg's ART papers. The human forgetting curve goes back to Ebbinghaus, *Über das Gedächtnis*, 1885; Wixted and Ebbesen, *On the Form of Forgetting*, Psychological Science 1991, is the standard case for the power-law fit. The claim that a gated state forgets along something like that shape is an argument from the spread of decay rates, not a measurement. Positions in the plasticity–retention figure and heights in the forgetting-curve figure are illustrative rather than measured, and the reused-key figure assumes $\beta = 1$ and unit-norm keys so that the update is a clean replacement.

---

{% include citation.html
  title="Every Sequence Model Is a Memory Algorithm"
  author="Renee Jia"
  journal="renee-jia.github.io"
  year="2026"
  url="https://renee-jia.github.io/research%20blog/ai%20research/every-sequence-model-is-a-memory-algorithm/"
  bibtex_key="reneejia2026memoryalgorithm"
%}
