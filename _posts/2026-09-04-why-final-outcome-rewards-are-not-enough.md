---
title: "Why Final-Outcome Rewards Are Not Enough for AI Agents"
date: 2026-09-04
categories:
  - Research Blog
  - AI Research
tags:
  - reinforcement learning
  - credit assignment
  - process reward models
  - counterfactual reasoning
  - multi-agent systems
  - LLM agents
excerpt: "An outcome reward tells you whether a trajectory worked. It doesn't say why. I think the interesting question is whether the process rewards we train on today measure a step's causal contribution, or mostly its resemblance to steps that show up in successful trajectories."
read_time: "15-18 min read"
layout: distill
toc: false
last_modified_at: 2026-09-04T08:00:00-08:00
---

*Outcome reward answers **whether**. Credit assignment answers **why**. I've been trying to understand how far apart those two questions have drifted.*

---

Reinforcement learning looks simple when you write it down. An agent takes a sequence of actions, receives a reward, and updates its policy to make high-reward trajectories more likely.

For modern agents, that abstraction leaves something open that I keep circling back to:

**When a long trajectory succeeds, which parts of it actually deserve the credit?**

Picture an agent solving a task in thirty steps. It tries a few approaches. It makes an incorrect assumption at step 8. It notices the inconsistency and recovers at step 17. It finds the key insight at step 23. It produces the right answer.

The environment returns $R = 1$.


<figure class="post-figure">
<svg viewBox="0 0 660 200" xmlns="http://www.w3.org/2000/svg" role="img" aria-label="A thirty-step trajectory returning a single scalar reward">
<defs><marker id="ah1" markerWidth="7" markerHeight="7" refX="6" refY="3" orient="auto"><path d="M0,0 L6,3 L0,6 z" fill="#9a7048"/></marker></defs>
<style>.lb{font-family:'Source Sans 3',system-ui,sans-serif;font-size:11px;fill:#8a7b6b}.lbs{font-family:'Source Sans 3',system-ui,sans-serif;font-size:11.5px;fill:#3d342b}</style>
<text x="18" y="22" class="lbs" font-weight="600">One trajectory, thirty decisions, one number.</text>
<text x="18" y="46" class="lb">&#964; = (s&#8321;,a&#8321;) &#8230; (s&#8323;&#8320;,a&#8323;&#8320;)</text>
<rect x="18" y="64" width="14" height="34" rx="2" fill="#f3ebe0" stroke="#e4d5c4" stroke-width="1"/><rect x="35" y="64" width="14" height="34" rx="2" fill="#f3ebe0" stroke="#e4d5c4" stroke-width="1"/><rect x="52" y="64" width="14" height="34" rx="2" fill="#f3ebe0" stroke="#e4d5c4" stroke-width="1"/><rect x="69" y="64" width="14" height="34" rx="2" fill="#f3ebe0" stroke="#e4d5c4" stroke-width="1"/><rect x="86" y="64" width="14" height="34" rx="2" fill="#f3ebe0" stroke="#e4d5c4" stroke-width="1"/><rect x="103" y="64" width="14" height="34" rx="2" fill="#f3ebe0" stroke="#e4d5c4" stroke-width="1"/><rect x="120" y="64" width="14" height="34" rx="2" fill="#f3ebe0" stroke="#e4d5c4" stroke-width="1"/><rect x="137" y="64" width="14" height="34" rx="2" fill="#f0dcd4" stroke="#a8543f" stroke-width="1"/><rect x="154" y="64" width="14" height="34" rx="2" fill="#f3ebe0" stroke="#e4d5c4" stroke-width="1"/><rect x="171" y="64" width="14" height="34" rx="2" fill="#f3ebe0" stroke="#e4d5c4" stroke-width="1"/><rect x="188" y="64" width="14" height="34" rx="2" fill="#f3ebe0" stroke="#e4d5c4" stroke-width="1"/><rect x="205" y="64" width="14" height="34" rx="2" fill="#f3ebe0" stroke="#e4d5c4" stroke-width="1"/><rect x="222" y="64" width="14" height="34" rx="2" fill="#f3ebe0" stroke="#e4d5c4" stroke-width="1"/><rect x="239" y="64" width="14" height="34" rx="2" fill="#f3ebe0" stroke="#e4d5c4" stroke-width="1"/><rect x="256" y="64" width="14" height="34" rx="2" fill="#f3ebe0" stroke="#e4d5c4" stroke-width="1"/><rect x="273" y="64" width="14" height="34" rx="2" fill="#f3ebe0" stroke="#e4d5c4" stroke-width="1"/><rect x="290" y="64" width="14" height="34" rx="2" fill="#e6e9dc" stroke="#6b7a52" stroke-width="1"/><rect x="307" y="64" width="14" height="34" rx="2" fill="#f3ebe0" stroke="#e4d5c4" stroke-width="1"/><rect x="324" y="64" width="14" height="34" rx="2" fill="#f3ebe0" stroke="#e4d5c4" stroke-width="1"/><rect x="341" y="64" width="14" height="34" rx="2" fill="#f3ebe0" stroke="#e4d5c4" stroke-width="1"/><rect x="358" y="64" width="14" height="34" rx="2" fill="#f3ebe0" stroke="#e4d5c4" stroke-width="1"/><rect x="375" y="64" width="14" height="34" rx="2" fill="#f3ebe0" stroke="#e4d5c4" stroke-width="1"/><rect x="392" y="64" width="14" height="34" rx="2" fill="#ecdfcd" stroke="#9a7048" stroke-width="1"/><rect x="409" y="64" width="14" height="34" rx="2" fill="#f3ebe0" stroke="#e4d5c4" stroke-width="1"/><rect x="426" y="64" width="14" height="34" rx="2" fill="#f3ebe0" stroke="#e4d5c4" stroke-width="1"/><rect x="443" y="64" width="14" height="34" rx="2" fill="#f3ebe0" stroke="#e4d5c4" stroke-width="1"/><rect x="460" y="64" width="14" height="34" rx="2" fill="#f3ebe0" stroke="#e4d5c4" stroke-width="1"/><rect x="477" y="64" width="14" height="34" rx="2" fill="#f3ebe0" stroke="#e4d5c4" stroke-width="1"/><rect x="494" y="64" width="14" height="34" rx="2" fill="#f3ebe0" stroke="#e4d5c4" stroke-width="1"/><rect x="511" y="64" width="14" height="34" rx="2" fill="#f3ebe0" stroke="#e4d5c4" stroke-width="1"/><text x="25" y="111" font-size="9" fill="#b5a594" text-anchor="middle" font-family="ui-monospace,monospace">1</text><text x="144" y="111" font-size="9" fill="#b5a594" text-anchor="middle" font-family="ui-monospace,monospace">8</text><text x="297" y="111" font-size="9" fill="#b5a594" text-anchor="middle" font-family="ui-monospace,monospace">17</text><text x="399" y="111" font-size="9" fill="#b5a594" text-anchor="middle" font-family="ui-monospace,monospace">23</text><text x="518" y="111" font-size="9" fill="#b5a594" text-anchor="middle" font-family="ui-monospace,monospace">30</text>
<path d="M 533 81.0 H 555" stroke="#9a7048" stroke-width="1.2" marker-end="url(#ah1)"/>
<rect x="559" y="67" width="52" height="28" rx="3" fill="#f4ece0" stroke="#9a7048"/>
<text x="585" y="86" font-family="ui-monospace,monospace" font-size="12.5" fill="#9a7048" text-anchor="middle">R = 1</text>
<line x1="144" y1="116" x2="144" y2="128" stroke="#a8543f" stroke-width="1"/>
<text x="144" y="140" class="lb" fill="#a8543f" text-anchor="middle">wrong assumption</text>
<line x1="297" y1="116" x2="297" y2="146" stroke="#6b7a52" stroke-width="1"/>
<text x="297" y="158" class="lb" fill="#6b7a52" text-anchor="middle">recovery</text>
<line x1="399" y1="116" x2="399" y2="128" stroke="#9a7048" stroke-width="1"/>
<text x="399" y="140" class="lb" fill="#9a7048" text-anchor="middle">key insight</text>
<text x="18" y="188" class="lb">The same scalar multiplies all thirty steps.</text>
</svg>
<figcaption>A successful thirty-step trajectory. The environment says <em>yes</em> exactly once.</figcaption>
</figure>

From the point of view of outcome reward, the whole trajectory was a success.

From the point of view of learning, that signal leaves a lot unsaid. Should the incorrect step at 8 be reinforced? Did the recovery at 17 matter, or would the agent have gotten there anyway? Was 23 the only step doing real work? Would the trajectory have succeeded with half of it deleted?

These are all versions of an old problem in a new setting: **credit assignment**.

I want to be careful about what's actually novel here, because a lot of good work already exists on it. The part I find genuinely open is narrower than "credit assignment is hard":

> Do the process rewards we train on measure a step's *causal contribution*, or largely its *resemblance* to steps that appear in successful trajectories — and does that difference change what the policy ends up learning?

Getting to a real answer means climbing a ladder, where each rung asks a more precise question than the one below it and costs more to compute:

$$\text{outcome} \rightarrow \text{process} \rightarrow \text{counterfactual} \rightarrow \text{interaction}$$

That ladder is the shape of what follows. I'll take the rungs in order: what outcome rewards leave out, what process rewards add and where I think they stop short, what a genuinely counterfactual notion of credit would require, and what happens to all of it once there's more than one agent in the room.

---

## The gap between whether and why

Take a trajectory

$$\tau = (s_1, a_1, s_2, a_2, \ldots, s_T, a_T)$$

with a final reward $R(\tau)$.

The reward tells us whether the trajectory succeeded. It doesn't directly say how much each action contributed. What we'd like is something closer to

$$C_t = \text{Contribution}(a_t \rightarrow R).$$

The distinction can read as a technicality, though I don't think it is one. An outcome reward answers *did the trajectory succeed?* Credit assignment asks *why did it succeed?*

The trouble is that a trajectory-level label structurally can't answer the second question. Say the policy produces $a_1, \ldots, a_{30}$ and receives $R = 1$. The outcome reward model observes one data point:

$$(a_1, \ldots, a_{30}) \rightarrow 1.$$

It doesn't observe the trajectory with $a_8$ removed, or with $a_8$ replaced by something else. Those are exactly the observations that would tell you whether $a_8$ mattered, and they're the ones a single label can never contain.

This is part of why outcome supervision can be efficient at *scoring* trajectories while staying weaker at *explaining* them. A decent verifier, a blunter teacher. Over many trajectories a lot of the noise does average out, which is why outcome-only RL works as well as it does. But the sample count needed scales poorly with horizon, and along the way some behavior gets reinforced mainly for having been nearby when things went right.

Arguably every RL method that works is doing some version of the conversion from *whether* to *why*: value functions, advantage estimates, baselines, GAE. A correlational version, as I'll argue later, but a version. Classical RL has worked on this for a long time. TD learning exists partly because propagating a terminal reward backwards through a value function carries more information than multiplying a whole trajectory by one scalar. RUDDER (Arjona-Medina et al., NeurIPS 2019) took on delayed reward directly, redistributing return toward the steps that shifted the expected outcome. Hindsight Credit Assignment (Harutyunyan et al., 2019) asked something close to: given that this outcome happened, how likely is it that this action was responsible?

So the problem isn't new. What seems different now is how much structure sits between the first action and the reward.

---

## Why agents widen the gap

**Trajectories got long.** A reasoning agent may emit thousands of tokens, run searches, execute code, call tools, read results, revise hypotheses, and retry failures before producing anything the environment can score. One scalar, arriving at the end, has to supervise all of it.

**Mistakes became recoverable.** Suppose the agent makes a bad inference at step 7,

$$a_7 = \text{wrong hypothesis},$$

then catches the inconsistency at step 12,

$$a_{12} = \text{correction}.$$

If the final answer is correct, reinforcing the trajectory uniformly reinforces both the mistake and the repair. This isn't purely hypothetical: self-correcting agents are trained on exactly the trajectories where errors happened and were survived. A learner that can't separate the error from the repair has at least some incentive to produce errors worth repairing.

**Some of the trace is probably redundant.** A reasoning trace tends to contain plausible-looking steps that may not do much work: restatements, hedges, verification that verifies nothing, a tool call whose output is never used again. Which gives the line I keep coming back to:

$$\text{present in a successful trajectory} \;\neq\; \text{causally responsible for success}.$$

Outcome-level RL treats those two as equivalent. Most of what follows is about how much that costs.

---

## Process rewards, and what they actually measure

The natural response is to stop issuing one reward per trajectory and start issuing rewards at intermediate steps. Instead of $R(\tau)$, learn

$$r_1, r_2, \ldots, r_T.$$

This is the motivation behind process reward models. *Let's Verify Step by Step* (Lightman et al., 2023) found that a reward model trained on step-level human labels beat an outcome-supervised one as a verifier re-ranking sampled solutions, solving 78% of a representative subset of MATH. Math-Shepherd (Wang et al., ACL 2024) then removed the need for human labels: from each intermediate step it samples $N$ completions and scores the step by whether they reach the correct answer, either as the fraction that succeed or, in the version they actually run, as whether *any* of them does. A step is good, in their phrasing, in proportion to "its potential to deduce the correct answer."

That definition is the interesting one, because it quietly changes what "good" means. Write the estimated quantity as

$$\Phi(h) = P(\text{success} \mid h),$$

and the natural per-step reward is its increment:

$$r_t \;\approx\; \Phi(h_{t+1}) - \Phi(h_t),$$

where $h_t$ is the history before action $a_t$. The reward becomes a measure of **progress**. An action earns credit if it moves the system somewhere success is more likely, which is much closer to what we want than a trajectory-level scalar.

This has been carried into agent settings, where it gets harder. Choudhury's AgentPRM (*Process Reward Models for LLM Agents: Practical Framework and Directions*, 2025) uses Monte Carlo rollouts in an actor-critic loop to train process rewards for interactive agents. A separate line of work also called AgentPRM (Xi et al., 2025) makes the underlying point explicit: in agent tasks, actions often have no clear-cut correctness, so they're better judged by proximity to the goal and the progress they've made. Both are reasonable, and both treat progress as the target quantity.

So: problem solved?

Not quite, I think. There's still daylight between

$$\text{this step looks good}$$

and

$$\text{this step caused the outcome to improve}.$$

The easy version of the worry is that a PRM is a learned function on step text, so it picks up whatever correlates with success in its training distribution, causes included and not. If successful solutions often contain *"let me verify this carefully,"* a model trained on them may learn to reward the phrase even where deleting the verification changes nothing.

Rollout-based scoring is more robust than that, since it's grounded in outcomes rather than appearance. But writing it as $\Phi$ makes two sharper things visible.

**It redistributes credit; it doesn't add evidence.** $\Phi(h_{t+1}) - \Phi(h_t)$ is potential-based shaping in the sense of Ng, Harada & Russell (1999). Summed along a trajectory it telescopes to $\Phi(h_T) - \Phi(h_0)$, so the total is pinned by the endpoints. That's a feature, not a bug: it's what keeps the shaping policy-invariant, and redistributing a fixed total is precisely the credit-assignment job. It does mean a progress-based PRM isn't contributing new information about the outcome. It's re-attributing an estimate the value function already holds.

**The estimate is correlational by construction.** $\Phi = P(\text{success} \mid h)$ is a value function of the current policy: an expectation conditional on *reaching* $h$ under $\pi$, not on *intervening* to put the agent there. Conditioning on $h_{t+1}$ tells you which states this policy tends to occupy after good steps. It can't isolate what the step itself did, because the policy that produced $h_{t+1}$ is the same one that continues from it. So a step that reliably *indicates* a competent trajectory, without *causing* its success, still scores well. Math-Shepherd's hard estimation sharpens this: scoring a step 1 if *any* of $N$ completions succeeds measures how reachable the answer is from that state, which is a property of the state more than a contribution of the step.

That's the gap I find interesting, and it points toward a more demanding notion of credit:

$$C_t = R(\tau) - R(\tau_{-t}),$$

where $\tau_{-t}$ is a counterfactual trajectory with step $t$ removed or replaced. The question becomes explicitly causal: would the outcome have changed without this action?

<figure class="post-figure">
<svg viewBox="0 0 660 300" xmlns="http://www.w3.org/2000/svg" role="img" aria-label="Outcome reward, process reward, and counterfactual credit over the same twelve-step trajectory">
<style>.lb{font-family:'Source Sans 3',system-ui,sans-serif;font-size:10.5px;fill:#8a7b6b}.lbs{font-family:'Source Sans 3',system-ui,sans-serif;font-size:11.5px;fill:#3d342b}</style>
<text x="14" y="30" class="lbs" font-weight="600">Outcome reward</text><text x="14" y="45" class="lb">every step gets R</text><line x1="114" y1="58.0" x2="623" y2="58.0" stroke="#e4d5c4" stroke-width="1"/><rect x="124" y="35.0" width="33" height="23.0" rx="2" fill="#e9ddd0" fill-opacity="1" stroke="#b5a594" stroke-width="0.8"/><rect x="166" y="35.0" width="33" height="23.0" rx="2" fill="#e9ddd0" fill-opacity="1" stroke="#b5a594" stroke-width="0.8"/><rect x="208" y="35.0" width="33" height="23.0" rx="2" fill="#e9ddd0" fill-opacity="1" stroke="#b5a594" stroke-width="0.8"/><rect x="250" y="35.0" width="33" height="23.0" rx="2" fill="#e9ddd0" fill-opacity="1" stroke="#b5a594" stroke-width="0.8"/><rect x="292" y="35.0" width="33" height="23.0" rx="2" fill="#e9ddd0" fill-opacity="1" stroke="#b5a594" stroke-width="0.8"/><rect x="334" y="35.0" width="33" height="23.0" rx="2" fill="#e9ddd0" fill-opacity="1" stroke="#b5a594" stroke-width="0.8"/><rect x="376" y="35.0" width="33" height="23.0" rx="2" fill="#e9ddd0" fill-opacity="1" stroke="#b5a594" stroke-width="0.8"/><rect x="418" y="35.0" width="33" height="23.0" rx="2" fill="#e9ddd0" fill-opacity="1" stroke="#b5a594" stroke-width="0.8"/><rect x="460" y="35.0" width="33" height="23.0" rx="2" fill="#e9ddd0" fill-opacity="1" stroke="#b5a594" stroke-width="0.8"/><rect x="502" y="35.0" width="33" height="23.0" rx="2" fill="#e9ddd0" fill-opacity="1" stroke="#b5a594" stroke-width="0.8"/><rect x="544" y="35.0" width="33" height="23.0" rx="2" fill="#e9ddd0" fill-opacity="1" stroke="#b5a594" stroke-width="0.8"/><rect x="586" y="35.0" width="33" height="23.0" rx="2" fill="#e9ddd0" fill-opacity="1" stroke="#b5a594" stroke-width="0.8"/>
<text x="14" y="126" class="lbs" font-weight="600">Process reward</text><text x="14" y="141" class="lb">how good does the step look?</text><line x1="114" y1="154.0" x2="623" y2="154.0" stroke="#e4d5c4" stroke-width="1"/><rect x="124" y="139.4" width="33" height="14.6" rx="2" fill="#e7e2d2" fill-opacity="1" stroke="#8a7b6b" stroke-width="0.8"/><rect x="166" y="133.1" width="33" height="20.9" rx="2" fill="#e7e2d2" fill-opacity="1" stroke="#8a7b6b" stroke-width="0.8"/><rect x="208" y="135.2" width="33" height="18.8" rx="2" fill="#e7e2d2" fill-opacity="1" stroke="#8a7b6b" stroke-width="0.8"/><rect x="250" y="128.9" width="33" height="25.1" rx="2" fill="#e7e2d2" fill-opacity="1" stroke="#8a7b6b" stroke-width="0.8"/><rect x="292" y="131.0" width="33" height="23.0" rx="2" fill="#e7e2d2" fill-opacity="1" stroke="#8a7b6b" stroke-width="0.8"/><rect x="334" y="145.6" width="33" height="8.4" rx="2" fill="#e7e2d2" fill-opacity="1" stroke="#8a7b6b" stroke-width="0.8"/><rect x="376" y="126.8" width="33" height="27.2" rx="2" fill="#e7e2d2" fill-opacity="1" stroke="#8a7b6b" stroke-width="0.8"/><rect x="418" y="124.7" width="33" height="29.3" rx="2" fill="#e7e2d2" fill-opacity="1" stroke="#8a7b6b" stroke-width="0.8"/><rect x="460" y="133.1" width="33" height="20.9" rx="2" fill="#e7e2d2" fill-opacity="1" stroke="#8a7b6b" stroke-width="0.8"/><rect x="502" y="120.6" width="33" height="33.4" rx="2" fill="#e7e2d2" fill-opacity="1" stroke="#8a7b6b" stroke-width="0.8"/><rect x="544" y="128.9" width="33" height="25.1" rx="2" fill="#e7e2d2" fill-opacity="1" stroke="#8a7b6b" stroke-width="0.8"/><rect x="586" y="122.7" width="33" height="31.3" rx="2" fill="#e7e2d2" fill-opacity="1" stroke="#8a7b6b" stroke-width="0.8"/>
<text x="14" y="222" class="lbs" font-weight="600">Counterfactual credit</text><text x="14" y="237" class="lb">would removing it have changed R?</text><line x1="114" y1="233.3" x2="623" y2="233.3" stroke="#e4d5c4" stroke-width="1"/><rect x="124" y="229.5" width="33" height="3.8" rx="2" fill="#f3ebe0" fill-opacity="0.75" stroke="#f3ebe0" stroke-width="0.8"/><rect x="166" y="219.2" width="33" height="14.0" rx="2" fill="#6b7a52" fill-opacity="0.75" stroke="#6b7a52" stroke-width="0.8"/><rect x="208" y="232.0" width="33" height="1.3" rx="2" fill="#f3ebe0" fill-opacity="0.75" stroke="#f3ebe0" stroke-width="0.8"/><rect x="250" y="224.3" width="33" height="8.9" rx="2" fill="#6b7a52" fill-opacity="0.75" stroke="#6b7a52" stroke-width="0.8"/><rect x="292" y="232.8" width="33" height="0.5" rx="2" fill="#f3ebe0" fill-opacity="0.75" stroke="#f3ebe0" stroke-width="0.8"/><rect x="334" y="233.3" width="33" height="11.5" rx="2" fill="#a8543f" fill-opacity="0.75"/><rect x="376" y="226.9" width="33" height="6.4" rx="2" fill="#6b7a52" fill-opacity="0.75" stroke="#6b7a52" stroke-width="0.8"/><rect x="418" y="210.3" width="33" height="23.0" rx="2" fill="#6b7a52" fill-opacity="0.75" stroke="#6b7a52" stroke-width="0.8"/><rect x="460" y="232.5" width="33" height="0.8" rx="2" fill="#f3ebe0" fill-opacity="0.75" stroke="#f3ebe0" stroke-width="0.8"/><rect x="502" y="211.6" width="33" height="21.7" rx="2" fill="#6b7a52" fill-opacity="0.75" stroke="#6b7a52" stroke-width="0.8"/><rect x="544" y="232.3" width="33" height="1.0" rx="2" fill="#f3ebe0" fill-opacity="0.75" stroke="#f3ebe0" stroke-width="0.8"/><rect x="586" y="225.6" width="33" height="7.7" rx="2" fill="#6b7a52" fill-opacity="0.75" stroke="#6b7a52" stroke-width="0.8"/>
<text x="124" y="290" class="lb">step 1</text>
<text x="619" y="290" class="lb" text-anchor="end">step 12</text>
</svg>
<figcaption>The same trajectory under three signals. Outcome reward is flat by construction. Process reward, as PRMs are actually trained, is a score in [0,1] that tracks how promising each step <em>looks</em>. Counterfactual credit is sparse and signed: most steps do little, a few carry the trajectory, and one actively hurt it.</figcaption>
</figure>

The third row is closer to what I'd want to train on. It's also considerably more expensive to obtain, though less impossible than I would have guessed a year ago.

---

## Load-bearing, decorative, and harmful-but-recovered

Define the counterfactual contribution of an action as

$$\Delta_t = R(\tau) - \mathbb{E}_{a'_t}\big[R(\tau_{a_t \leftarrow a'_t})\big],$$

the drop in expected return from replacing $a_t$ with a resampled alternative. (What "resampled" should mean turns out to matter a great deal, and I'll come back to it.) Actions then sort roughly into three classes. I've found the naming useful, mostly because it makes the failure mode easier to see.

**Load-bearing actions.** $\Delta_t \gg 0$. Remove or perturb the action and the success rate falls noticeably. The key insight at step 23; the tool call that surfaced the constraint everything else depended on.

**Decorative actions.** $\Delta_t \approx 0$. The action sits in the trace with little effect on the outcome. Restatements, hedges, verification that verified nothing, the search whose results were ignored. Not harmful so much as unearned.

**Harmful-but-recovered actions.** $\Delta_t < 0$. The action made things worse and later steps paid for the repair, like the wrong assumption at step 8. It lowered the success probability; the trajectory succeeded anyway.

I should say that this vocabulary isn't mine alone, and the empirical picture is further along than I've implied so far. The decorative/load-bearing distinction is the framing of [my own recent work with Di Mu](/about/) on how task difficulty shapes the causal role of chain-of-thought (TMLR, 2026), and related taxonomies have appeared elsewhere; several groups now separate reasoning that is genuinely necessary from reasoning that is scaffolding or ornament. Li et al. (2026) report that latent answer commitment and explicit answer arrival align on only about 62% of steps across nine models, with most of the mismatch coming from traces that continue producing deliberative-looking text after the answer has effectively stabilized. Their reading is that much post-commitment text isn't load-bearing.

So "some of the trace is decorative" is better described as an emerging finding than an open question, at least for static chain-of-thought.

What outcome-level RL does with these three classes is the part I'd emphasize. It sees them as members of the same successful trajectory and multiplies all of them by the same $+1$. A process reward model does better on the first two, and may still struggle with the third, since a harmful-but-recovered step often *looks* locally reasonable, which is usually why the agent took it.

Of the three, the decorative class is the one I'd worry about first, because it's where reward hacking has the most room. If decorative steps draw positive credit, one cheap way to raise expected reward is to produce more of them: verbose reasoning that doesn't help, tool calls made for the appearance of diligence, self-checks that check nothing. Each is a fairly rational response to a signal that rewards presence rather than contribution.

---

## What "it could have been different" actually means

Everything above leans on a counterfactual, so it's worth asking what one actually requires.

For an action:

$$C_t = R(\tau) - \mathbb{E}_{a'_t}\big[R(\tau \mid \mathrm{do}(a_t = a'_t))\big].$$

For an agent:

$$C_i = R(\tau) - \mathbb{E}\big[R(\tau \mid \mathrm{do}(A_i = A'_i))\big].$$

Writing $\mathrm{do}$ doesn't settle things by itself, though. Interventional credit is only well-defined once you specify the replacement distribution, and the choice of "what would the agent have done instead" carries a lot of weight. Replace the step with a sample from the current policy and you measure contribution relative to the agent's own competence; a uniformly strong agent may show near-zero credit almost everywhere. Replace it with a null action and you're closer to measuring necessity, but you've changed the trajectory distribution and pushed the continuation off-policy.

When I first drafted this piece I wanted to say there's no canonical choice. I think that's now too strong. Macar et al. (2025) make a fairly persuasive case for one: fix the segment, resample only the subsequent text, and measure the effect across the distribution of continuations. They report that artificial edits, replacing a step's content, produce small and unstable effects relative to resampling, and they add a resilience measure that repeatedly resamples so deleted content doesn't simply reappear downstream. That last detail strikes me as the one people underestimate. A step can look unnecessary because the model regenerates its content a few tokens later, which is a statement about the policy's redundancy rather than the step's irrelevance.

There's also a cheaper route. Mesnard et al. (ICML 2021) try to get the variance reduction without the rollouts by learning a hindsight model (predict the counterfactual rather than sample it), which relocates the question to whether the learned model has captured causation or correlation.

So the replacement distribution is more settled than I assumed, at least for single-model chain-of-thought. Whether the same procedure transfers to agent trajectories is much less clear to me. A step there may be an irreversible tool call, a file write, or a message another agent has already acted on. You can resample a paragraph. Resampling an action whose side effects have already propagated is a different kind of problem.

---

## When there's more than one agent

Once an agent can call tools, spawn subtasks, or talk to other agents, credit assignment stops being purely temporal. Consider a system that runs

$$\text{Planner} \rightarrow \text{Researcher} \rightarrow \text{Critic} \rightarrow \text{Solver}$$

and produces the right answer, $R = 1$. Who earned it?

There are several nested levels at which "who did the work" is a meaningful question:

<figure class="post-figure">
<svg viewBox="0 0 660 288" xmlns="http://www.w3.org/2000/svg" role="img" aria-label="Credit hierarchy from token up to team">
<defs><marker id="ah3" markerWidth="7" markerHeight="7" refX="5" refY="3" orient="auto"><path d="M0,0 L6,3 L0,6 z" fill="#b5a594"/></marker></defs>
<style>.lb{font-family:'Source Sans 3',system-ui,sans-serif;font-size:10.5px;fill:#8a7b6b}.lbs{font-family:'Source Sans 3',system-ui,sans-serif;font-size:11.5px;fill:#3d342b}</style>
<text x="26" y="22" class="lbs" font-weight="600">Six levels at which &#8220;who did the work?&#8221; is a real question.</text>
<rect x="26" y="40" width="140" height="26" rx="3" fill="#f3ebe0" stroke="#e4d5c4"/><text x="38" y="57" class="lbs">token</text><text x="176" y="57" class="lb">10&#179;&#8211;10&#8309; per task</text><path d="M 36 66 V 75" stroke="#b5a594" stroke-width="1" marker-end="url(#ah3)"/><rect x="26" y="76" width="196" height="26" rx="3" fill="#f7f1e7" stroke="#e4d5c4"/><text x="38" y="93" class="lbs">reasoning step</text><text x="232" y="93" class="lb">10&#8211;10&#178;</text><path d="M 36 102 V 111" stroke="#b5a594" stroke-width="1" marker-end="url(#ah3)"/><rect x="26" y="112" width="252" height="26" rx="3" fill="#f3ebe0" stroke="#e4d5c4"/><text x="38" y="129" class="lbs">tool action</text><text x="288" y="129" class="lb">10&#8211;10&#178;</text><path d="M 36 138 V 147" stroke="#b5a594" stroke-width="1" marker-end="url(#ah3)"/><rect x="26" y="148" width="308" height="26" rx="3" fill="#f7f1e7" stroke="#e4d5c4"/><text x="38" y="165" class="lbs">agent turn</text><text x="344" y="165" class="lb">10&#8211;10&#178;</text><path d="M 36 174 V 183" stroke="#b5a594" stroke-width="1" marker-end="url(#ah3)"/><rect x="26" y="184" width="364" height="26" rx="3" fill="#f3ebe0" stroke="#e4d5c4"/><text x="38" y="201" class="lbs">agent</text><text x="400" y="201" class="lb">2&#8211;10</text><path d="M 36 210 V 219" stroke="#b5a594" stroke-width="1" marker-end="url(#ah3)"/><rect x="26" y="220" width="420" height="26" rx="3" fill="#f0e6d6" stroke="#9a7048"/><text x="38" y="237" class="lbs">team</text><text x="456" y="237" class="lb">1</text>
<path d="M 470 233 H 440" stroke="#9a7048" stroke-width="1.2" marker-end="url(#ah3b)"/>
<defs><marker id="ah3b" markerWidth="7" markerHeight="7" refX="6" refY="3" orient="auto"><path d="M0,0 L6,3 L0,6 z" fill="#9a7048"/></marker></defs>
<text x="476" y="237" font-family="ui-monospace,monospace" font-size="12" fill="#9a7048">R = 1 arrives here</text>
<text x="26" y="276" class="lb">The reward is observed once, at the team level. Every level above it has to be paid out of that one number.</text>
</svg>
<figcaption>Credit has to be divided at every level, and the levels don't decompose cleanly into each other.</figcaption>
</figure>

A correct final answer doesn't imply every node in this hierarchy contributed positively, or at all. Temporal credit assignment becomes **hierarchical** credit assignment, and the levels interact: a load-bearing agent can contain decorative turns, and an otherwise decorative agent can contribute one load-bearing token.

At the agent level the failure gets easier to see. Three agents collaborate on an answer:

$$A, B, C \rightarrow y, \qquad R(y) = 1.$$

The simplest thing is to give everyone the outcome:

$$r_A = r_B = r_C = 1.$$

That assignment quietly claims all three contributed equally, which seems unlikely to hold often. Suppose $A$ found the solution, $B$ restated $A$'s argument in different words, and $C$ caught and repaired a crucial error. A more informative decomposition might be

$$r_A = 0.6, \qquad r_B = 0.0, \qquad r_C = 0.4.$$

Even that may be wrong, since $A$ and $C$ might only be valuable together; $C$'s repair is worth little without $A$'s solution to repair. Then the meaningful quantity isn't individual credit but an interaction term:

$$I_{AC} = R(A, C) - R(A) - R(C),$$

which is the top rung of the ladder. Credit may belong less to agents than to *interactions between* agents. This is roughly where the problem stops looking like RL bookkeeping and starts looking like cooperative game theory, which is where multi-agent RL went some time ago. COMA (Foerster et al., AAAI 2018) computes a counterfactual baseline that marginalizes out one agent's action while holding the others fixed; the Shapley-value line distributes credit by averaging marginal contributions over coalitions. Both try to answer *what would have happened without you*, in a setting where "without you" isn't well-defined until you say what replaces you — the same question as the previous section, now one level up.

The LLM version is younger. MAPoRL (Park et al., ACL 2025) co-trains multiple language models with RL, scoring the final collaborative output with a verifier that also adds incentives for corrective and persuasive discussion. I think that's a genuine step toward trainable collaboration. It also raises a question it doesn't try to settle: *why* should a correction be rewarded? Because it looks corrective? Because the final answer improved? Because another agent changed its behavior in response? Because it was causally necessary? Those four definitions likely produce different learning dynamics. Rewarding the appearance of correction seems like a plausible route to agents that manufacture things to correct.

---

## Why this shows up in the gradient

It'd be easy to file all of this under interpretability, but I think it also changes optimization.

The trajectory-level policy gradient looks roughly like

$$\nabla J = \mathbb{E}\Big[R \sum_t \nabla \log \pi(a_t \mid h_t)\Big].$$

Every action is multiplied by the same scalar, so the learning signal for a step carries no information about that step.

With step-level credit:

$$\nabla J = \mathbb{E}\Big[\sum_t C_t \nabla \log \pi(a_t \mid h_t)\Big].$$

Different actions now receive different signals, and the estimator's variance improves with the quality of $C_t$.

For multi-agent systems:

$$\nabla J = \mathbb{E}\Big[\sum_i \sum_t C_{i,t} \nabla \log \pi_i(a_{i,t} \mid h_{i,t})\Big].$$

The quality of $C_{i,t}$ plausibly decides whether the system learns collaboration or mostly learns behaviors that co-occurred with success. This is also where the decorative-step problem could become self-reinforcing: if $C_t > 0$ for decorative steps, the policy raises their probability, future trajectories contain more of them, and the credit model sees more decorative steps inside successful trajectories to learn from. That's a loop that compounds rather than washes out, though I should be clear this is a mechanism I find plausible rather than one I've measured.

---

## What I'd still like to know

Given the work above, the honest version of "open questions" is narrower than I'd have written a year ago. Static chain-of-thought is being measured; the agent setting looks much less charted.

**Does the decorative fraction hold up in agent trajectories?** The CoT evidence suggests a substantial share of steps aren't load-bearing. Agent trajectories differ in ways that could push it either direction: tool calls return real information, which argues for fewer decorative steps, but they're also cheap to issue and look diligent, which argues for more. I don't know of a clean measurement, and the resampling machinery doesn't transfer directly once actions have side effects.

**Do process reward models rank decorative steps above load-bearing ones?** Score steps with a PRM, score the same steps by resampling-based ablation, and look at the rank correlation. Any systematic gap is the distance between *looks good* and *did work*, made concrete. My guess is the disagreement concentrates on steps that most resemble good reasoning: verification, restatement, explicit self-doubt. But that's a guess, and I'd want to see it fail as readily as succeed.

**Does credit-shaped training shorten traces?** If decorative steps are being reinforced by outcome-level RL, then training the same model with counterfactual credit should shorten its traces without hurting accuracy. That seems like a reasonably clean prediction, and a falsifiable one.

**Where does agent-level credit have to be estimated rather than computed?** With three agents you can ablate exhaustively. With ten you can't, and you're back to sampling coalitions or learning a model, with the correlation-versus-causation exposure that implies.

---

## Credit, not only reward

The usual framing in RL is *what reward should we optimize?* For agents that run long, use tools, correct themselves, and work alongside other agents, I think a second question is becoming similarly load-bearing:

**How should observed reward be distributed across the computation that produced it?**

A final reward tells us whether the system worked. A good credit mechanism tries to say what made it work. While trajectories were short, the two questions had nearly the same answer, and conflating them cost little. That seems less true now, and the further apart they drift, the more of what an agent learns may come from steps that were simply present when things went right.

I'd be glad to be wrong about the size of that gap. It's the kind of claim that measurement should settle rather than argument, and the questions above are my best attempt at saying what measurement would do it.

---

## Notes

References:

- Arjona-Medina, Gillhofer, Widrich, Unterthiner, Brandstetter & Hochreiter, *RUDDER: Return Decomposition for Delayed Rewards*, NeurIPS 2019 — [arXiv:1806.07857](https://arxiv.org/abs/1806.07857)
- Harutyunyan et al., *Hindsight Credit Assignment*, NeurIPS 2019 — [arXiv:1912.02503](https://arxiv.org/abs/1912.02503)
- Foerster, Farquhar, Afouras, Nardelli & Whiteson, *Counterfactual Multi-Agent Policy Gradients* (COMA), AAAI 2018 — [arXiv:1705.08926](https://arxiv.org/abs/1705.08926)
- Mesnard et al., *Counterfactual Credit Assignment in Model-Free Reinforcement Learning*, ICML 2021 — [arXiv:2011.09464](https://arxiv.org/abs/2011.09464)
- Ng, Harada & Russell, *Policy Invariance Under Reward Transformations: Theory and Application to Reward Shaping*, ICML 1999
- Lightman et al., *Let's Verify Step by Step*, 2023 — [arXiv:2305.20050](https://arxiv.org/abs/2305.20050)
- Wang et al., *Math-Shepherd: Verify and Reinforce LLMs Step-by-step without Human Annotations*, ACL 2024 — [arXiv:2312.08935](https://arxiv.org/abs/2312.08935)
- Choudhury, *Process Reward Models for LLM Agents: Practical Framework and Directions* (AgentPRM), 2025 — [arXiv:2502.10325](https://arxiv.org/abs/2502.10325)
- Xi et al., *AgentPRM: Process Reward Models for LLM Agents via Step-Wise Promise and Progress*, 2025 — [arXiv:2511.08325](https://arxiv.org/abs/2511.08325)
- Park, Han, Guo, Ozdaglar, Zhang & Kim, *MAPoRL: Multi-Agent Post-Co-Training for Collaborative Large Language Models with Reinforcement Learning*, ACL 2025 — [arXiv:2502.18439](https://arxiv.org/abs/2502.18439)
- Macar, Bogdan, Rajamanoharan & Nanda, *Thought Branches: Interpreting LLM Reasoning Requires Resampling*, 2025 — [arXiv:2510.27484](https://arxiv.org/abs/2510.27484)
- Li, Yang, Hazarika, Mehta & Onoue, *When Reasoning Traces Become Performative: Step-Level Evidence that Chain-of-Thought Is an Imperfect Oversight Channel*, 2026 — [arXiv:2605.11746](https://arxiv.org/abs/2605.11746)

Two papers share the AgentPRM name; they're separate pieces of work and both are listed above.
