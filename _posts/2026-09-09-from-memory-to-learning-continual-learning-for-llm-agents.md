---
title: "From Memory to Learning: What Continual Learning for LLM Agents Would Actually Look Like"
date: 2026-09-09
categories:
  - Research Blog
  - AI Research
tags:
  - continual learning
  - agent memory
  - experience learning
  - self-evolving agents
  - credit assignment
  - LLM agents
excerpt: "An agent that solves the same class of problem a thousand times should be better at it by the end. Most deployed agents aren't. I think the reason is that we've been building memory when the actual problem is consolidation: deciding which experience deserves to become a skill, a rule, or a weight update, and which deserves to be forgotten."
read_time: "15-18 min read"
layout: distill
toc: false
last_modified_at: 2026-09-09T08:00:00-08:00
---

*Memory answers **what happened**. Learning answers **what to do differently**. I've been trying to work out how much of the gap between them is engineering and how much is an open research problem.*

---

Large language models are oddly static objects.

We pretrain them, align them, ship them, and then drop them into environments that never hold still. Users change. Tools change. An API that took a bearer token last month wants OAuth this month. And the agents built on top of these models generate enormous amounts of experience: thousands of trajectories full of successes, failures, dead ends, workarounds, and the occasional genuine discovery.

Almost none of it survives.

A deployed coding agent can solve the same category of bug three hundred times in a quarter and be no better at it in April than it was in January. Every run starts from the same weights and the same prompt. Whatever it figured out last time went into a log file that nobody, human or model, will read again.

So the question I want to spend this post on is:

> **What would it mean for an LLM agent to actually learn from its own experience, continuously, without a human in the loop retraining the model?**

It isn't the same question as "how do we give agents memory," though the two get conflated constantly. Memory, it turns out, is the easy half. The hard half is turning repeated experience into something the agent can reuse, and that half looks a lot like continual learning with one extra problem bolted on: deciding which experiences deserved the credit in the first place.

---

## What continual learning meant before agents

The classical setting is a sequence of learning problems:

$$\mathcal{D}_1 \rightarrow \mathcal{D}_2 \rightarrow \cdots \rightarrow \mathcal{D}_T$$

The learner should get good at $\mathcal D_t$ without getting worse at $\mathcal D_1, \ldots, \mathcal D_{t-1}$. The failure mode with a name is *catastrophic forgetting*: gradient steps on the current distribution overwrite whatever made the earlier distributions work. The remedies fall into three families, and every one of them reappears later in a different costume.

| Family | Mechanism | The agent-world version |
|---|---|---|
| Replay | Keep a buffer of old examples and mix them into new training | Retrieval-augmented experience; "experience RAG" |
| Regularization | Penalize movement on parameters that mattered for old tasks | Write only repeatedly confirmed experience into weights; keep the rest external |
| Parameter isolation | Give each task its own subset of parameters | Sparse memory layers, per-skill adapters, skill libraries outside the model |

For LLMs, "continual" can mean adapting at any of several stages. Shi et al.'s 2024 survey splits it into continual pretraining, domain-adaptive pretraining, and continual fine-tuning, and separately into vertical continuity (general capabilities narrowing to specific ones) and horizontal continuity (adapting across time and domains). The taxonomy is mostly a reminder that this is at least three problems with different data, costs, and failure modes.

Agents add a fourth, and to me it's the most interesting of the lot. In every setting above, the datasets arrive from outside. An agent's don't. **An agent manufactures its own training distribution by acting.**

---

## Agents write their own training distribution

Picture a coding agent working on a set of repositories for a few months. Every task produces a trajectory,

$$\tau = (s_0, a_0, o_1, a_1, o_2, \ldots, a_T, r),$$

with states, reasoning, tool calls, observations, and at the end some outcome $r$. After a hundred thousand tasks, the agent has been the sole witness to a great deal of useful information. Which search strategies pay off in this codebase. Which tool calls fail, and why. Which planning mistakes recur, which recovery moves work, what this particular team's conventions are.

The intuition everyone has is

$$\text{experience} \rightarrow \text{better future behavior}.$$

What actually happens in most deployed systems is

$$\text{experience} \rightarrow \text{logs}.$$

<figure class="post-figure">
<svg viewBox="0 0 660 230" xmlns="http://www.w3.org/2000/svg" role="img" aria-label="Two fates for an agent's experience: it becomes logs, or it changes future behavior">
<defs><marker id="f1a" markerWidth="7" markerHeight="7" refX="6" refY="3" orient="auto"><path d="M0,0 L6,3 L0,6 z" fill="#b5a594"/></marker><marker id="f1b" markerWidth="7" markerHeight="7" refX="6" refY="3" orient="auto"><path d="M0,0 L6,3 L0,6 z" fill="#9a7048"/></marker></defs>
<style>.lb{font-family:'Source Sans 3',system-ui,sans-serif;font-size:10.5px;fill:#8a7b6b}.lbs{font-family:'Source Sans 3',system-ui,sans-serif;font-size:11.5px;fill:#3d342b}.mono{font-family:ui-monospace,monospace;font-size:11px;fill:#3d342b}</style>
<text x="18" y="22" class="lbs" font-weight="600">Where does an agent's experience go?</text>
<!-- policy box -->
<rect x="18" y="80" width="96" height="40" rx="3" fill="#f3ebe0" stroke="#e4d5c4"/>
<text x="66" y="97" class="lbs" text-anchor="middle">policy</text>
<text x="66" y="111" class="mono" text-anchor="middle">&#960;<tspan font-size="8" dy="3">&#952;</tspan></text>
<!-- trajectories -->
<path d="M 114 100 H 160" stroke="#b5a594" stroke-width="1.2" marker-end="url(#f1a)"/>
<g>
<rect x="166" y="60" width="176" height="18" rx="2" fill="#f7f1e7" stroke="#e4d5c4"/><text x="172" y="73" class="mono">&#964;&#8321; &#8230; success</text>
<rect x="166" y="82" width="176" height="18" rx="2" fill="#f7f1e7" stroke="#e4d5c4"/><text x="172" y="95" class="mono">&#964;&#8322; &#8230; fail (auth)</text>
<rect x="166" y="104" width="176" height="18" rx="2" fill="#f7f1e7" stroke="#e4d5c4"/><text x="172" y="117" class="mono">&#964;&#8323; &#8230; success, 40 steps</text>
<rect x="166" y="126" width="176" height="18" rx="2" fill="#f7f1e7" stroke="#e4d5c4"/><text x="172" y="139" class="mono">&#964;&#8324; &#8230; timeout</text>
<text x="241" y="160" class="lb" text-anchor="middle">&#8942;  100,000 tasks later</text>
</g>
<!-- upper branch: logs -->
<path d="M 342 84 C 380 84, 400 50, 440 50" stroke="#b5a594" stroke-width="1.2" fill="none" marker-end="url(#f1a)"/>
<rect x="446" y="34" width="110" height="32" rx="3" fill="#f7f1e7" stroke="#e4d5c4" stroke-dasharray="3 2"/>
<text x="501" y="54" class="lbs" text-anchor="middle">logs</text>
<text x="501" y="80" class="lb" text-anchor="middle">stored, rarely read,</text>
<text x="501" y="93" class="lb" text-anchor="middle">never changes &#960;</text>
<!-- lower branch: learning loop -->
<path d="M 342 120 C 380 120, 400 150, 440 150" stroke="#9a7048" stroke-width="1.2" fill="none" marker-end="url(#f1b)"/>
<rect x="446" y="134" width="110" height="32" rx="3" fill="#ecdfcd" stroke="#9a7048"/>
<text x="501" y="154" class="lbs" text-anchor="middle">consolidation</text>
<path d="M 501 166 V 200 H 66 V 122" stroke="#9a7048" stroke-width="1.2" fill="none" marker-end="url(#f1b)"/>
<text x="276" y="213" class="lb" text-anchor="middle" fill="#9a7048">memory, skills, or weights: something that changes what &#960; does next time</text>
<text x="566" y="148" class="lb">the loop most</text>
<text x="566" y="161" class="lb">agents leave open</text>
</svg>
<figcaption>The same trajectories can end up as logs or as learning. Most deployed agents today take the top branch.</figcaption>
</figure>

That figure is the whole post in one picture. The upper branch is where nearly every production agent lives today. The lower branch is what continual learning for agents would mean, and the interesting design work is entirely inside the box labeled *consolidation*: not whether to remember, but what to remember, in what form, and when to stop.

---

## Memory is not learning

The distinction has been sharpening in the literature, so it's worth walking through the systems that established it. Each one moved experience a step further from raw history, and each step exposed a new problem.

**Generative Agents** (Park et al., UIST 2023) gave 25 simulated characters a memory stream of natural-language observations, retrieved by recency, importance, and relevance, and periodically synthesized into higher-level *reflections*. Their ablation showed that removing reflection measurably hurt believability, an early hint that a raw stream on its own isn't enough. **MemGPT** (Packer et al., 2023) treated the problem as an operating-systems problem, paging information between a fixed context window and external storage. It's the cleanest statement of the *storage* problem and, deliberately, says nothing about learning.

**Reflexion** (Shinn et al., NeurIPS 2023) is the first system on this list that improves through experience. After a failed attempt the agent writes a verbal self-reflection, keeps it in an episodic buffer, and conditions the next attempt on it. No gradient touches the weights, and it still reached 91% pass@1 on HumanEval against GPT-4's 80% at the time. The buffer is capped at the last three reflections, which I read as a hint that this kind of memory saturates fast.

**Voyager** (Wang et al., 2023) stored something different in kind: a library of executable skills, each a verified program that can be retrieved and composed later. Competence compounds because the stored artifact is itself a capability. In Minecraft it collected 3.3× more unique items and hit key tech-tree milestones up to 15.3× faster than the prior state of the art.

**ExpeL** (Zhao et al., AAAI 2024) made the comparison step explicit. It extracts natural-language insights by contrasting a failed attempt with a successful one on the same task, and by looking across a set of successes for what they share. It's the earliest system I know of that treats *pairs* of trajectories, rather than individual ones, as the unit of learning.

More recent systems scale the same idea up. **Agent Workflow Memory** (Wang et al., ICML 2025) induces reusable workflows from past trajectories, with 24.6% and 51.1% relative gains in success rate on Mind2Web and WebArena. **ReasoningBank** (Ouyang et al., ICLR 2026) distills strategies from both self-judged successes and failures, reporting up to 20% relative improvement and up to 16% fewer interaction steps on web and software-engineering benchmarks. **EvolveR** (Wu et al., ICML 2026) closes the loop: an offline stage distills trajectories into strategic principles, an online stage retrieves them while acting, and an RL step updates the policy from the resulting trajectories.

| System | What it stores | Unit of learning | Weights change? |
|---|---|---|---|
| Generative Agents (2023) | Observations, reflections | Observations, aggregated into reflections | No |
| MemGPT (2023) | Paged context | None (storage only) | No |
| Reflexion (2023) | Verbal self-reflections | One failed trajectory | No |
| Voyager (2023) | Executable skills | One verified program | No |
| ExpeL (2024) | Natural-language insights | Success/failure pairs; sets of successes | No |
| Agent Workflow Memory (2025) | Induced workflows | Sets of trajectories | No |
| ReasoningBank (2026) | Reasoning strategies | Success and failure trajectories | No |
| EvolveR (2026) | Strategic principles + policy | Trajectory repository | Yes (RL) |

Luo et al.'s survey in Findings of ACL 2026 formalizes this arc as three stages, Storage, Reflection, and Experience, and treats the third as the current frontier. That matches my reading, with one addition: the third stage is where the problem stops being about memory at all.

Two things stand out from the table. Almost every system keeps the weights frozen, so "learning" has mostly meant better prompting from a growing file. And the unit of learning has been sliding from one trajectory toward many. That slide is the part that matters. The question stops being *how do we give agents more memory* and becomes:

> **How does an agent turn repeated experience into reusable competence, and how does it know when it has?**

---

## Four places experience can live

I find it clarifying to separate continual learning in agents into four levels by where the experience ends up. Each level compresses more, generalizes further, and is harder to undo.

<figure class="post-figure">
<svg viewBox="0 0 660 300" xmlns="http://www.w3.org/2000/svg" role="img" aria-label="Four levels of continual agent learning, from episodic storage to parametric consolidation">
<defs><marker id="f2a" markerWidth="7" markerHeight="7" refX="6" refY="3" orient="auto"><path d="M0,0 L6,3 L0,6 z" fill="#9a7048"/></marker></defs>
<style>.lb{font-family:'Source Sans 3',system-ui,sans-serif;font-size:10.5px;fill:#8a7b6b}.lbs{font-family:'Source Sans 3',system-ui,sans-serif;font-size:11.5px;fill:#3d342b}.mono{font-family:ui-monospace,monospace;font-size:10.5px;fill:#9a7048}</style>
<text x="18" y="22" class="lbs" font-weight="600">Four places experience can live.</text>
<!-- axes -->
<line x1="60" y1="250" x2="620" y2="250" stroke="#e4d5c4" stroke-width="1"/>
<line x1="60" y1="250" x2="60" y2="50" stroke="#e4d5c4" stroke-width="1"/>
<text x="340" y="272" class="lb" text-anchor="middle">more compressed, more reusable &#8594;</text>
<text x="40" y="150" class="lb" text-anchor="middle" transform="rotate(-90 40 150)">harder to revise &#8594;</text>
<!-- steps -->
<rect x="80" y="196" width="118" height="44" rx="3" fill="#f7f1e7" stroke="#e4d5c4"/>
<text x="139" y="213" class="lbs" text-anchor="middle">1. Episodic storage</text>
<text x="139" y="229" class="mono" text-anchor="middle">M = {&#964;&#8321;, &#8230;, &#964;<tspan font-size="8" dy="2">N</tspan>}</text>
<rect x="218" y="150" width="118" height="44" rx="3" fill="#f3ebe0" stroke="#e4d5c4"/>
<text x="277" y="167" class="lbs" text-anchor="middle">2. Reflection</text>
<text x="277" y="183" class="mono" text-anchor="middle">m<tspan font-size="8" dy="2">i</tspan><tspan dy="-2"> = g(&#964;</tspan><tspan font-size="8" dy="2">i</tspan><tspan dy="-2">)</tspan></text>
<rect x="356" y="104" width="118" height="44" rx="3" fill="#ecdfcd" stroke="#d9c3a5"/>
<text x="415" y="121" class="lbs" text-anchor="middle">3. Abstraction</text>
<text x="415" y="137" class="mono" text-anchor="middle">z = f(&#964;&#8321;, &#8230;, &#964;<tspan font-size="8" dy="2">k</tspan>)</text>
<rect x="494" y="58" width="118" height="44" rx="3" fill="#e6d3bb" stroke="#9a7048"/>
<text x="553" y="75" class="lbs" text-anchor="middle">4. Parametric</text>
<text x="553" y="91" class="mono" text-anchor="middle">&#952; &#8592; &#952; + &#916;&#952;(z)</text>
<!-- arrows -->
<path d="M 198 218 H 210 V 172 H 214" stroke="#9a7048" stroke-width="1.1" fill="none" marker-end="url(#f2a)"/>
<path d="M 336 172 H 348 V 126 H 352" stroke="#9a7048" stroke-width="1.1" fill="none" marker-end="url(#f2a)"/>
<path d="M 474 126 H 486 V 80 H 490" stroke="#9a7048" stroke-width="1.1" fill="none" marker-end="url(#f2a)"/>
<!-- annotations under -->
<text x="139" y="188" class="lb" text-anchor="middle" font-size="9.5">retrieve similar past runs</text>
<text x="277" y="142" class="lb" text-anchor="middle" font-size="9.5">one lesson per trajectory</text>
<text x="405" y="96" class="lb" text-anchor="middle" font-size="9.5">one rule, many trajectories</text>
<text x="553" y="50" class="lb" text-anchor="middle" font-size="9.5">default behavior, no retrieval</text>
<!-- example systems -->
<text x="139" y="290" class="lb" text-anchor="middle" font-size="9.5" fill="#b5a594">experience RAG</text>
<text x="277" y="290" class="lb" text-anchor="middle" font-size="9.5" fill="#b5a594">Reflexion</text>
<text x="415" y="290" class="lb" text-anchor="middle" font-size="9.5" fill="#b5a594">Voyager skills, ExpeL insights</text>
<text x="553" y="290" class="lb" text-anchor="middle" font-size="9.5" fill="#b5a594">fine-tuning, SEAL</text>
</svg>
<figcaption>Each step up trades revisability for reuse. Most current systems live on the first two rungs.</figcaption>
</figure>

**Level 1: episodic storage.** Keep the trajectories, $M = \lbrace \tau_1, \ldots, \tau_N \rbrace$, and retrieve similar ones when a new task arrives. This is experience RAG, and it works. It also scales badly: $|M| = O(N)$ while the useful signal in $M$ grows much more slowly. At some point the retriever starts surfacing stale, contradictory, or merely lucky trajectories, and the difficulty quietly shifts from remembering to deciding what deserves to be remembered.

**Level 2: reflection.** Summarize each trajectory into a lesson. "Tried API A, failed on auth, tried API B, succeeded" becomes "when credentials are unavailable, prefer API B." This is Reflexion's move, and it's cheap. But one trajectory is one sample, and the lesson can be confidently wrong: the agent may decide that calling tool X first caused the success when the success came from a later step. I'll call this *spurious consolidation*. It's the memory-side cousin of a problem I wrote about [last week](/research%20blog/ai%20research/why-final-outcome-rewards-are-not-enough/): reinforcing whatever was present when things went right.

**Level 3: abstraction.** Look at $k$ trajectories together and extract what they share, $z = f(\tau_1, \ldots, \tau_k)$, where $z$ might be a rule, a procedure, a program, or a fragment of policy. Voyager's skills and ExpeL's insights are early versions; EvolveR's principles are a recent one. This is where the system starts to look like continual learning rather than note-taking.

**Level 4: parametric consolidation.** Sufficiently stable experience migrates into the weights:

$$M_{\text{episodic}} \rightarrow M_{\text{semantic}} \rightarrow \theta.$$

The analogy to human memory consolidation is obvious, and mostly right. Recent experience is episodic, repeated patterns become semantic knowledge, and stable knowledge becomes default behavior that no longer needs retrieval. Hu et al.'s recent survey makes a similar move, treating token-level, parametric, and latent memory as three *forms* of one thing rather than three research areas.

The mechanics of level 4 have moved quickly since early 2025. **SEAL** (Zweiger et al., NeurIPS 2025) has the model generate its own fine-tuning data and update directives, with an RL outer loop that rewards self-edits by the downstream performance of the updated model. **Sparse memory finetuning** (Lin et al., 2025) updates only the memory slots a new fact activates strongly; on their setup, full fine-tuning on new facts cut NaturalQuestions F1 by 89% and LoRA by 71%, while sparse updates cost an 11% drop with the same amount learned. **Titans** and **Nested Learning** (Behrouz et al., both NeurIPS 2025) make a long-term memory that updates at test time part of the architecture itself.

None of these is an agent paper. But they're the first credible answers to *how* to write experience into weights without wrecking what's there, which makes the *what* and *when* questions urgent. The rest of this post is about those: when experience should move up a level, what a piece of experience is worth, when it stops being worth that, what failed runs contribute, and where the lessons actually live. I don't have answers to most of these. I'm fairly sure they're the right questions.

---

## When should an experience become a weight update?

Start with the last gate, because it's the one with the clearest cost. Today the decision to move experience into parameters is made globally: either everything stays in external memory forever, or you periodically fine-tune on whatever has accumulated. Both are wrong for the same reason. The right answer differs per experience.

Imagine each candidate experience gets a consolidation score,

$$C(\tau) = f(\text{utility}, \text{novelty}, \text{frequency}, \text{confidence}, \text{stability}),$$

and gets routed by it:

$$\tau \;\rightarrow\; \begin{cases} \text{discard} \\ \text{episodic memory} \\ \text{abstract skill} \\ \text{parameter update} \end{cases}$$

One-off, low-confidence experiences stay episodic, where they're cheap to revise. Patterns that recur get abstracted. Only knowledge that has proven stable across time and context earns a gradient step, where it becomes fast and default but expensive to retract.

This is a *memory-to-weights routing problem*, and I haven't seen it posed as a learning problem in its own right. The experiment I keep sketching gives an agent a long task stream with recurring strategies, one-off solutions, noisy feedback, and a few deliberate distribution shifts, then compares storing everything, periodic replay fine-tuning, and a learned consolidation policy on future success, forgetting, memory size, and adaptation speed after each shift. Notice that none of this is about *how* to update the model. The question underneath is which information deserves to enter the model at all.

---

## Not every successful experience is worth remembering

That question hides an assumption: that we can tell what an experience is worth. Mostly we can't. This is the part I've spent the most time on, and the one that changed my mind about what agent memory is for.

An agent takes 40 actions and succeeds. Which parts should be remembered?

$$a_1, a_2, \ldots, a_{40} \;\rightarrow\; R = 1$$

Current systems store the whole trajectory, an LLM-written summary of it, or the final reflection. All three assume a successful trajectory is made of useful steps. It isn't. Some steps were unnecessary. Some were harmful and later recovered from. Some had nothing to do with the outcome. A memory extracted from the whole thing inherits all of that.

So memory formation contains a credit assignment problem, and I'd argue a harder one than the action-level version, because the artifact being credited will be reused across *future* tasks that don't exist yet. Define the value of a memory fragment $m_i$ as

$$V(m_i) = \mathbb{E}\big[R \mid m_i \text{ available}\big] - \mathbb{E}\big[R \mid m_i \text{ removed}\big],$$

with the expectation over the future tasks the fragment might be retrieved for. In practice you'd approximate it by counterfactual replay: hold out one fragment, re-run a batch of held-out tasks, see what changes.

<figure class="post-figure">
<svg viewBox="0 0 660 258" xmlns="http://www.w3.org/2000/svg" role="img" aria-label="Counterfactual ablation of memory fragments: removing each fragment and measuring future success">
<style>.lb{font-family:'Source Sans 3',system-ui,sans-serif;font-size:10.5px;fill:#8a7b6b}.lbs{font-family:'Source Sans 3',system-ui,sans-serif;font-size:11.5px;fill:#3d342b}.mono{font-family:ui-monospace,monospace;font-size:10.5px;fill:#3d342b}</style>
<text x="18" y="22" class="lbs" font-weight="600">Which memory fragments are actually doing the work?</text>
<text x="18" y="40" class="lb">One successful 40-step trajectory, chopped into five candidate memories. Remove each, re-run held-out tasks.</text>
<!-- header row -->
<text x="18" y="70" class="lb">fragment</text>
<text x="250" y="70" class="lb">success rate on held-out tasks, fragment removed</text>
<line x1="250" y1="76" x2="600" y2="76" stroke="#e4d5c4"/>
<!-- baseline marker -->
<line x1="530" y1="78" x2="530" y2="212" stroke="#9a7048" stroke-width="1" stroke-dasharray="3 2"/>
<text x="530" y="222" class="lb" text-anchor="middle" fill="#9a7048">all fragments kept: 0.80</text>
<!-- rows: bar length = 250 + rate*350 -->
<g>
<text x="18" y="96" class="mono">m&#8321; read the API docs first</text>
<rect x="250" y="86" width="126" height="12" rx="1" fill="#f0dcd4" stroke="#a8543f"/>
<text x="600" y="96" class="mono" fill="#a8543f" text-anchor="end">0.36</text>
</g>
<g>
<text x="18" y="122" class="mono">m&#8322; retried with backoff</text>
<rect x="250" y="112" width="266" height="12" rx="1" fill="#f3ebe0" stroke="#e4d5c4"/>
<text x="600" y="122" class="mono" text-anchor="end">0.76</text>
</g>
<g>
<text x="18" y="148" class="mono">m&#8323; "verified carefully"</text>
<rect x="250" y="138" width="280" height="12" rx="1" fill="#f3ebe0" stroke="#e4d5c4"/>
<text x="600" y="148" class="mono" text-anchor="end">0.80</text>
</g>
<g>
<text x="18" y="174" class="mono">m&#8324; hard-coded token from run 3</text>
<rect x="250" y="164" width="301" height="12" rx="1" fill="#e6e9dc" stroke="#6b7a52"/>
<text x="600" y="174" class="mono" fill="#6b7a52" text-anchor="end">0.86</text>
</g>
<g>
<text x="18" y="200" class="mono">m&#8325; used /v1/search endpoint</text>
<rect x="250" y="190" width="287" height="12" rx="1" fill="#f3ebe0" stroke="#e4d5c4"/>
<text x="600" y="200" class="mono" text-anchor="end">0.82</text>
</g>
<text x="18" y="236" class="lb">Illustrative numbers. Removing m&#8321; hurts, so it was load-bearing. Removing m&#8324; helps: the trajectory</text>
<text x="18" y="249" class="lb">succeeded despite it, not because of it. m&#8323; changes nothing either way.</text>
</svg>
<figcaption>Presence in a successful trajectory is not the same as causing the success. Counterfactual replay separates the two, one fragment at a time.</figcaption>
</figure>

The figure is illustrative, but the shape is what I'd expect. Most fragments do nothing. One or two are load-bearing. At least one is actively harmful: it was present in a successful run, got stored because the run succeeded, and now misleads future runs. Under the usual pipeline all five would have been stored with equal confidence.

This changes what "good memory" means. Not *successful trajectory, therefore good memory*, but *causally useful segment, therefore good memory*. It also connects agent memory to the credit-assignment literature directly rather than by analogy. The same machinery that asks *would this outcome have happened without step 23?* can ask *would future outcomes be worse without memory item 23?* Counterfactual replay over future tasks is pricier than resampling within one trajectory, but the structure is the same, and the cheap approximations should carry over.

The one-line version is the section title. What I like about the claim is that it's testable with counterfactual memory ablation on benchmarks that already exist, no new framework and no new model required. It's a much narrower claim than "agents need better memory," and the narrow version is the one I've come to believe.

---

## Forgetting is a feature

Credit assignment tells you what a memory was worth when it was formed. It says nothing about whether it's still worth that a month later.

Continual-learning research has treated forgetting as the enemy since the late 1980s, when catastrophic interference first got its name. For agents, perfect retention is just as dangerous. Suppose an API changes. The agent's memory says *use `/v1/search`*; the environment now wants `/v2/search`. Every retrieval of that memory makes the agent worse, and the more confidently it was consolidated, the worse it gets. The objective was never $\min \text{forgetting}$. It was closer to

$$\max\; \text{retained useful knowledge} \;-\; \lambda\, \text{retained obsolete knowledge},$$

and the second term has been ignored because in the classical setting nothing ever became obsolete.

The mechanical version of this is old. MemoryBank (Zhong et al., AAAI 2024) decays memory strength on an Ebbinghaus-style forgetting curve and strengthens a memory each time it's recalled. Each memory could carry

$$m_i = (\text{content}, \text{confidence}, \text{timestamp}, \text{usage}, \text{success rate}),$$

with a retrieval weight that decays as $w_i(t) = w_i(0)\,e^{-\lambda_i t}$ and gets bumped by successful reuse. That handles staleness. It doesn't handle contradiction, and contradiction is the hard case: two memories that can't both be true, and the agent has to decide which one the world currently agrees with. Hu et al.'s survey names this the stability-plasticity dilemma, and MemoryAgentBench (Hu, Wang & McAuley, ICLR 2026) makes *selective forgetting* one of its four required competencies, reporting that no current method masters all four.

The version I'd want to build treats forgetting not as deletion but as **belief revision**: the memory system maintains something like a posterior over which of its own entries are still true, and retrieval is weighted by it.

---

## Failures deserve their own memory

Everything so far has quietly assumed the experience came from a run that worked. Most experience-learning systems make the same assumption. Failures, if they're kept at all, become a "don't do that again" note.

But success and failure carry different information. A success says *this strategy worked here*. A failure says *this region of the policy space is unsafe*, which is a statement about a neighborhood rather than a point, and my guess is that it generalizes better for exactly that reason. Instead of one memory bank, keep two,

$$M^{+} = \text{successful strategies}, \qquad M^{-} = \text{failure modes},$$

and let the policy condition on both:

$$a_t = \pi\big(s_t, \operatorname{retrieve}(M^{+}), \operatorname{retrieve}(M^{-})\big).$$

ReasoningBank is the clearest recent system built on this asymmetry. ExpeL had the sharper version two years earlier: put a failed and a successful trajectory on the same task side by side, a matched pair $(\tau^+, \tau^-)$, and ask what differs. There's a parametric version too. *Agent Learning via Early Experience* (Zhang et al., ICML 2026) trains the policy on the states its own suboptimal actions lead to, with no reward signal at all.

The matched-pair question is the one I'd push on. Two trajectories that are identical up to step $t$ and then split localize the decisive decision far more precisely than any single-trajectory reflection can. It's contrastive learning at the trajectory level, and it produces a much cleaner artifact than "here is what I learned" prose generated from one run.

---

## The lesson lives between trajectories, not inside one

Matched pairs are a special case of something more general. Most systems summarize trajectories independently, $f(\tau_i) \rightarrow m_i$. But the abstractions worth having usually don't live in any single trajectory.

Trajectory A: searched the documentation first, succeeded. Trajectory B: guessed the API signature, failed. Trajectory C: searched the repository for usage examples, succeeded. The useful lesson isn't in any of the three. It's

> *When facing an unfamiliar interface, retrieve authoritative information about it before attempting execution.*

That's cross-trajectory abstraction, $f(\tau_1, \ldots, \tau_k) \rightarrow \text{principle}$, which Luo et al. single out, along with proactive exploration, as the defining mechanism of their Experience stage. One way to formalize it is clustering: find latent strategies $z_j$ shared by many trajectories, then test whether retrieving $z_j$ generalizes better than retrieving the demonstrations that produced it.

I'd expect the answer to be yes, with a caveat that matters. Abstraction is also where spurious consolidation gets amplified. One bad lesson from one trajectory pollutes one memory. One bad principle abstracted from twenty trajectories pollutes everything downstream, with the authority of having been "confirmed" twenty times. Which is why credit assignment has to come before abstraction, not after it.

---

## Learning needs a compute budget

Every mechanism above costs something: counterfactual replay, contradiction checks, clustering, gradient steps. A deployed agent may generate millions of interactions, and you can't fine-tune after each one. You probably shouldn't even reflect after each one. Continual agent learning is a resource allocation problem before it's a learning problem.

Say an update costs $C_{\text{update}}$ and its expected future improvement is $\Delta V$. Consolidate only when $\Delta V > C_{\text{update}}$. Since $\Delta V$ scales with how often the knowledge will be reused and $C_{\text{update}}$ with how deep the write goes, this alone suggests a layered architecture.

<figure class="post-figure">
<svg viewBox="0 0 660 270" xmlns="http://www.w3.org/2000/svg" role="img" aria-label="Three nested loops for continual agent learning at different timescales">
<defs><marker id="f4a" markerWidth="7" markerHeight="7" refX="6" refY="3" orient="auto"><path d="M0,0 L6,3 L0,6 z" fill="#9a7048"/></marker></defs>
<style>.lb{font-family:'Source Sans 3',system-ui,sans-serif;font-size:10.5px;fill:#8a7b6b}.lbs{font-family:'Source Sans 3',system-ui,sans-serif;font-size:11.5px;fill:#3d342b}.mono{font-family:ui-monospace,monospace;font-size:10.5px;fill:#9a7048}</style>
<text x="18" y="22" class="lbs" font-weight="600">Three loops, three timescales, three costs.</text>
<!-- timeline -->
<line x1="40" y1="60" x2="620" y2="60" stroke="#e4d5c4" stroke-width="1"/>
<g>
<!-- fast loop ticks: every task -->
<rect x="40" y="52" width="2" height="16" fill="#b5a594"/><rect x="60" y="52" width="2" height="16" fill="#b5a594"/><rect x="80" y="52" width="2" height="16" fill="#b5a594"/><rect x="100" y="52" width="2" height="16" fill="#b5a594"/><rect x="120" y="52" width="2" height="16" fill="#b5a594"/><rect x="140" y="52" width="2" height="16" fill="#b5a594"/><rect x="160" y="52" width="2" height="16" fill="#b5a594"/><rect x="180" y="52" width="2" height="16" fill="#b5a594"/><rect x="200" y="52" width="2" height="16" fill="#b5a594"/><rect x="220" y="52" width="2" height="16" fill="#b5a594"/><rect x="240" y="52" width="2" height="16" fill="#b5a594"/><rect x="260" y="52" width="2" height="16" fill="#b5a594"/><rect x="280" y="52" width="2" height="16" fill="#b5a594"/><rect x="300" y="52" width="2" height="16" fill="#b5a594"/><rect x="320" y="52" width="2" height="16" fill="#b5a594"/><rect x="340" y="52" width="2" height="16" fill="#b5a594"/><rect x="360" y="52" width="2" height="16" fill="#b5a594"/><rect x="380" y="52" width="2" height="16" fill="#b5a594"/><rect x="400" y="52" width="2" height="16" fill="#b5a594"/><rect x="420" y="52" width="2" height="16" fill="#b5a594"/><rect x="440" y="52" width="2" height="16" fill="#b5a594"/><rect x="460" y="52" width="2" height="16" fill="#b5a594"/><rect x="480" y="52" width="2" height="16" fill="#b5a594"/><rect x="500" y="52" width="2" height="16" fill="#b5a594"/><rect x="520" y="52" width="2" height="16" fill="#b5a594"/><rect x="540" y="52" width="2" height="16" fill="#b5a594"/><rect x="560" y="52" width="2" height="16" fill="#b5a594"/><rect x="580" y="52" width="2" height="16" fill="#b5a594"/><rect x="600" y="52" width="2" height="16" fill="#b5a594"/>
</g>
<!-- medium loop ticks -->
<circle cx="140" cy="60" r="5" fill="#ecdfcd" stroke="#9a7048"/><circle cx="300" cy="60" r="5" fill="#ecdfcd" stroke="#9a7048"/><circle cx="460" cy="60" r="5" fill="#ecdfcd" stroke="#9a7048"/><circle cx="620" cy="60" r="5" fill="#ecdfcd" stroke="#9a7048"/>
<!-- slow loop -->
<rect x="454" y="40" width="12" height="40" rx="2" fill="none" stroke="#a8543f" stroke-width="1.2"/>
<text x="40" y="88" class="lb">time &#8594;</text>
<!-- legend rows -->
<g>
<rect x="40" y="110" width="2" height="14" fill="#b5a594"/>
<text x="56" y="121" class="lbs" font-weight="600">Fast loop</text><text x="130" y="121" class="lb">every task</text>
<text x="250" y="121" class="lb">retrieve &#8594; act &#8594; record the trajectory</text>
<text x="560" y="121" class="mono">cost: inference</text>
</g>
<g>
<circle cx="41" cy="156" r="5" fill="#ecdfcd" stroke="#9a7048"/>
<text x="56" y="160" class="lbs" font-weight="600">Medium loop</text><text x="150" y="160" class="lb">every N tasks</text>
<text x="250" y="160" class="lb">cluster, dedupe, resolve contradictions, abstract</text>
<text x="560" y="160" class="mono">cost: LLM calls</text>
</g>
<g>
<rect x="36" y="187" width="10" height="16" rx="2" fill="none" stroke="#a8543f" stroke-width="1.2"/>
<text x="56" y="199" class="lbs" font-weight="600">Slow loop</text><text x="130" y="199" class="lb">rarely, and only when &#916;V &gt; C<tspan font-size="8" dy="2">update</tspan></text>
<text x="250" y="216" class="lb">move stable, high-value experience into &#952; (SFT / LoRA / RL)</text>
<text x="560" y="199" class="mono">cost: training</text>
</g>
<text x="40" y="252" class="lb">Fast memory, slow learning. The interesting design questions are the gates between the loops.</text>
</svg>
<figcaption>Three timescales for consolidating experience. The gates between loops are where a consolidation policy would live.</figcaption>
</figure>

The fast loop runs every task: retrieve, act, record. The medium loop runs every $N$ tasks: cluster, deduplicate, resolve contradictions, abstract. The slow loop runs rarely, moving stable, high-value experience into parameters with SFT, a LoRA adapter, or RL. **Fast memory, slow learning.** Practical systems will probably end up looking like this whether or not anyone designs them to, because the cost structure forces it. What's not forced is the design of the gates between loops, and that's where everything in the previous five sections lives.

---

## Measure the lifetime, not the task

Suppose someone built all of this. How would we know it worked? Nearly every agent benchmark reports performance on a fixed set of tasks from a cold start. A continual agent should be evaluated over its whole lifetime, $P_1, P_2, \ldots, P_T$, and the numbers that matter are derivatives, not levels.

| What to measure | Quantity | What "good" looks like |
|---|---|---|
| Positive transfer | $P_{t+k}$ on related tasks vs $P_t$ | Rising |
| Harmful forgetting | $P_{t+k}$ on still-valid old tasks vs $P_t$ | Flat |
| Adaptation speed | $\partial P / \partial N_{\text{experience}}$ | High, especially after a shift |
| Memory efficiency | $\Delta P / \lvert M \rvert$ | High; memory shouldn't grow linearly with tasks |
| Compute efficiency | $\Delta P / \text{training FLOPs}$ | High; most gains should come from the fast loop |

The memory benchmarks that exist already show how much cold-start metrics hide. LongMemEval (Wu et al., ICLR 2025) found commercial chat assistants and long-context models losing about 30% accuracy on information spread across sustained interactions, and LoCoMo (Maharana et al., ACL 2024) found both long context and retrieval well short of humans on conversations averaging 600 turns. But those are still mostly recall benchmarks. What I'd want is one where the agent is *supposed* to get better over a long stream, where some of the stream's regularities change midway, and where the score is the shape of the learning curve. That's the only measurement that separates an agent that learns from one that merely remembers.

---

## From continual learning to self-evolving agents

Put together, this is a shift in what an agent *is*. A static agent is an LLM plus tools plus a prompt. A memory-augmented agent adds a store. A genuinely self-improving agent is

$$\boxed{\text{Agent}_{t+1} = \operatorname{Learn}\big(\text{Agent}_t, \text{Experience}_t\big)}$$

where the learner is part of the architecture, not an offline process someone runs in a training cluster.

The surveys have started calling this *self-evolving* or *lifelong* agents. Gao et al.'s TMLR survey organizes the field around what to evolve, when, and how; Fang et al. frame it as a feedback loop over inputs, agent, environment, and optimizer. Both are fine taxonomies, and neither spends much time on the questions above. Most of the field's energy has gone into *how to evolve*, and much less into *what evidence should justify evolving*. That's the property I'd hold a self-evolving agent to. Not autonomy, which is cheap, but:

> **Does interacting with the world make the agent better at interacting with the world?**

---

## Maybe next

I started to summarize this literature review expecting the interesting problems to be about memory: how to store more, retrieve better, summarize more faithfully. I've come out of it thinking the memory part is further along than I expected, and that what's unsolved sits one step downstream. Knowing what to keep. Knowing what a kept thing is worth. Knowing when it has stopped being true. None of those is a retrieval problem.

The frame that stuck with me is the loop, not the store:

$$\begin{aligned}
\text{Interact} &\rightarrow \text{Evaluate} \rightarrow \text{Assign credit} \rightarrow \text{Store} \\
&\rightarrow \text{Abstract} \rightarrow \text{Consolidate} \rightarrow \text{Forget} \rightarrow \text{Interact again}
\end{aligned}$$

Memory is one box in that loop. Every recent system I looked at improves that one box. Almost none of them touch the boxes on either side of it, and I now think that's why so many of them plateau: a better memory of experience that was never credited, never contradicted, and never allowed to expire is still a better log.

---

## Notes

References:

- Shi et al., *Continual Learning of Large Language Models: A Comprehensive Survey*, 2024 — [arXiv:2404.16789](https://arxiv.org/abs/2404.16789)
- Park et al., *Generative Agents: Interactive Simulacra of Human Behavior*, UIST 2023 — [arXiv:2304.03442](https://arxiv.org/abs/2304.03442)
- Packer et al., *MemGPT: Towards LLMs as Operating Systems*, 2023 — [arXiv:2310.08560](https://arxiv.org/abs/2310.08560)
- Shinn et al., *Reflexion: Language Agents with Verbal Reinforcement Learning*, NeurIPS 2023 — [arXiv:2303.11366](https://arxiv.org/abs/2303.11366)
- Wang et al., *Voyager: An Open-Ended Embodied Agent with Large Language Models*, TMLR 2024 — [arXiv:2305.16291](https://arxiv.org/abs/2305.16291)
- Zhao et al., *ExpeL: LLM Agents Are Experiential Learners*, AAAI 2024 — [arXiv:2308.10144](https://arxiv.org/abs/2308.10144)
- Wang et al., *Agent Workflow Memory*, ICML 2025 — [arXiv:2409.07429](https://arxiv.org/abs/2409.07429)
- Zhong et al., *MemoryBank: Enhancing Large Language Models with Long-Term Memory*, AAAI 2024 — [arXiv:2305.10250](https://arxiv.org/abs/2305.10250)
- Ouyang et al., *ReasoningBank: Scaling Agent Self-Evolving with Reasoning Memory*, ICLR 2026 — [arXiv:2509.25140](https://arxiv.org/abs/2509.25140)
- Wu et al., *EvolveR: Self-Evolving LLM Agents through an Experience-Driven Lifecycle*, ICML 2026 — [arXiv:2510.16079](https://arxiv.org/abs/2510.16079)
- Zhang et al., *Agent Learning via Early Experience*, ICML 2026 — [arXiv:2510.08558](https://arxiv.org/abs/2510.08558)
- Zweiger et al., *Self-Adapting Language Models* (SEAL), NeurIPS 2025 — [arXiv:2506.10943](https://arxiv.org/abs/2506.10943)
- Lin et al., *Continual Learning via Sparse Memory Finetuning*, 2025 — [arXiv:2510.15103](https://arxiv.org/abs/2510.15103)
- Behrouz, Zhong & Mirrokni, *Titans: Learning to Memorize at Test Time*, NeurIPS 2025 — [arXiv:2501.00663](https://arxiv.org/abs/2501.00663)
- Behrouz et al., *Nested Learning: The Illusion of Deep Learning Architectures*, NeurIPS 2025 — [arXiv:2512.24695](https://arxiv.org/abs/2512.24695)
- Hu et al., *Memory in the Age of AI Agents*, 2025 — [arXiv:2512.13564](https://arxiv.org/abs/2512.13564)
- Luo et al., *From Storage to Experience: A Survey on the Evolution of LLM Agent Memory Mechanisms*, Findings of ACL 2026 — [arXiv:2605.06716](https://arxiv.org/abs/2605.06716), [doi:10.18653/v1/2026.findings-acl.2069](https://aclanthology.org/2026.findings-acl.2069/)
- Gao et al., *A Survey of Self-Evolving Agents: What, When, How, and Where to Evolve on the Path to Artificial Super Intelligence*, TMLR 2026 — [arXiv:2507.21046](https://arxiv.org/abs/2507.21046)
- Fang et al., *A Comprehensive Survey of Self-Evolving AI Agents: A New Paradigm Bridging Foundation Models and Lifelong Agentic Systems*, 2025 — [arXiv:2508.07407](https://arxiv.org/abs/2508.07407)
- Maharana et al., *Evaluating Very Long-Term Conversational Memory of LLM Agents* (LoCoMo), ACL 2024 — [arXiv:2402.17753](https://arxiv.org/abs/2402.17753)
- Wu et al., *LongMemEval: Benchmarking Chat Assistants on Long-Term Interactive Memory*, ICLR 2025 — [arXiv:2410.10813](https://arxiv.org/abs/2410.10813)
- Hu, Wang & McAuley, *Evaluating Memory in LLM Agents via Incremental Multi-Turn Interactions* (MemoryAgentBench), ICLR 2026 — [arXiv:2507.05257](https://arxiv.org/abs/2507.05257)

The ReasoningBank numbers quoted above are from the ICLR camera-ready (v2 on arXiv). The first arXiv version reported a larger figure, up to 34.2% relative improvement, for its test-time scaling variant; I've used the more conservative published numbers. The counterfactual-ablation figure in the credit-assignment section is illustrative, not measured.

---

{% include citation.html
  title="From Memory to Learning: What Continual Learning for LLM Agents Would Actually Look Like"
  author="Renee Jia"
  journal="renee-jia.github.io"
  year="2026"
  url="https://renee-jia.github.io/research%20blog/ai%20research/from-memory-to-learning-continual-learning-for-llm-agents/"
  bibtex_key="reneejia2026memorytolearning"
%}
