---
title: "Is Reinforcement Learning Fundamentally a Game of Distributions?"
date: 2026-11-22
research_kind: open-problem
categories:
  - Research Blog
  - AI Research
tags:
  - reinforcement learning
  - post-training
  - policy gradient
  - reward design
  - LLM training
excerpt: "Better reasoning, better answers, the right format, following instructions. We list these as separate things RL gives a model. Underneath, they may all be the same move: post-training takes the pretrained model's distribution and shifts it toward the outputs people want."
read_time: "8-10 min read"
layout: distill
toc: false
last_modified_at: 2026-11-22T08:00:00-08:00
---

A language model is usually trained in stages. Pretraining teaches it to predict text. Mid-training, when there is one, takes it deeper into a particular domain and specific tasks. Post-training then shapes how it behaves, typically with supervised fine-tuning on demonstrations first and reinforcement learning after that.

<figure class="post-figure post-figure--doodle">
<svg viewBox="0 0 660 215" xmlns="http://www.w3.org/2000/svg" role="img" aria-label="A doodle of the training pipeline as four boxes in a row: pretraining, mid-training on a particular domain and specific tasks, supervised fine-tuning and reinforcement learning. A bracket above the last two is labeled post-training, and the reinforcement learning box is highlighted as the subject of this post.">
<defs>
<filter id="roughP" x="-5%" y="-5%" width="110%" height="110%"><feTurbulence type="fractalNoise" baseFrequency="0.03" numOctaves="2" seed="31" result="n"/><feDisplacementMap in="SourceGraphic" in2="n" scale="2.2" xChannelSelector="R" yChannelSelector="G"/></filter>
</defs>
<style>.hw{font-family:'Caveat','Cormorant Garamond',cursive;fill:#4e4338}.hw-s{font-family:'Caveat','Cormorant Garamond',cursive;fill:#8a7b6b}</style>
<g filter="url(#roughP)" fill="none" stroke-linecap="round" stroke-linejoin="round">
<path d="M 12 70 H 150 V 126 H 12 Z" stroke="#4e4338" stroke-width="1.7" fill="#f3ebe0"/>
<path d="M 155 98 H 173 M 165 91 L 174 98 L 165 105" stroke="#4e4338" stroke-width="1.7"/>
<path d="M 178 70 H 316 V 126 H 178 Z" stroke="#4e4338" stroke-width="1.7" fill="#f3ebe0"/>
<path d="M 321 98 H 339 M 331 91 L 340 98 L 331 105" stroke="#4e4338" stroke-width="1.7"/>
<path d="M 344 70 H 482 V 126 H 344 Z" stroke="#4e4338" stroke-width="1.7" fill="#f3ebe0"/>
<path d="M 487 98 H 505 M 497 91 L 506 98 L 497 105" stroke="#4e4338" stroke-width="1.7"/>
<path d="M 510 70 H 648 V 126 H 510 Z" stroke="#9a7048" stroke-width="2.1" fill="#ecdfcd"/>
<path d="M 344 60 V 48 H 648 V 60" stroke="#8a7b6b" stroke-width="1.4"/>
</g>
<text x="81" y="105" class="hw" font-size="20" text-anchor="middle">pretraining</text>
<text x="247" y="105" class="hw" font-size="20" text-anchor="middle">mid-training</text>
<text x="413" y="94" class="hw" font-size="20" text-anchor="middle">supervised</text>
<text x="413" y="115" class="hw" font-size="20" text-anchor="middle">fine-tuning</text>
<text x="579" y="94" class="hw" font-size="20" text-anchor="middle">reinforcement</text>
<text x="579" y="115" class="hw" font-size="20" text-anchor="middle">learning</text>
<text x="496" y="38" class="hw-s" font-size="19" text-anchor="middle">post-training</text>
<text x="81" y="150" class="hw-s" font-size="16" text-anchor="middle">predict the next token</text>
<text x="81" y="169" class="hw-s" font-size="16" text-anchor="middle">on a lot of text</text>
<text x="247" y="150" class="hw-s" font-size="16" text-anchor="middle">go deeper into</text>
<text x="247" y="169" class="hw-s" font-size="16" text-anchor="middle">a particular domain</text>
<text x="247" y="188" class="hw-s" font-size="16" text-anchor="middle">+ specific tasks</text>
<text x="413" y="150" class="hw-s" font-size="16" text-anchor="middle">imitate good</text>
<text x="413" y="169" class="hw-s" font-size="16" text-anchor="middle">demonstrations</text>
<text x="579" y="150" class="hw-s" font-size="16" text-anchor="middle">try, get scored,</text>
<text x="579" y="169" class="hw-s" font-size="16" text-anchor="middle">adjust</text>
<text x="650" y="200" class="hw" font-size="18" text-anchor="end" style="fill:#9a7048">this post is about this step</text>
</svg>
<figcaption>Where RL sits in the training of a language model.</figcaption>
</figure>

A pretrained model is a predictor. Given some text, it gives every possible continuation a probability, and those probabilities reflect the text it was trained on. That is not the same as what a person wants from it. Ask it a question, and a likely continuation might be another question, or an answer in the wrong form.

Post-training is there to close this gap. It uses demonstrations, human preferences as in RLHF, and verifiable rewards as in RLVR, where a checker decides whether a math answer or a piece of code is right. These rewards have become more specific and more carefully built over time. They now target reasoning ability as well as the correctness of the answer, the format, and whether the instruction was followed.

When people describe the result, they usually give a list. The model reasons more accurately. Its answers are more helpful. It follows instructions more closely, and it sticks to the format it was asked for.

These sound like four different abilities, and they are often measured on four different benchmarks. But the more I look at how post-training is set up, the more they look like one thing to me.

Each of the training signals above does the same job. It takes the probabilities the pretrained model started with and moves them toward the outputs people actually want. The reward says which responses we prefer, and the optimization tries to make the model more likely to produce them. Whether the reward is about correctness or about JSON brackets, the thing being changed is the same.

<figure class="post-figure post-figure--doodle">
<svg viewBox="0 0 660 250" xmlns="http://www.w3.org/2000/svg" role="img" aria-label="A doodle with four goals on the left, better reasoning, better answers, the right format and following instructions. Arrows from all four lead into one bubble labeled reward, which leads to a hill of probability that has shifted from a dashed pretrained position to a solid post-trained position.">
<defs>
<filter id="roughG" x="-5%" y="-5%" width="110%" height="110%"><feTurbulence type="fractalNoise" baseFrequency="0.03" numOctaves="2" seed="11" result="n"/><feDisplacementMap in="SourceGraphic" in2="n" scale="2.2" xChannelSelector="R" yChannelSelector="G"/></filter>
</defs>
<style>.hw{font-family:'Caveat','Cormorant Garamond',cursive;fill:#4e4338}.hw-s{font-family:'Caveat','Cormorant Garamond',cursive;fill:#8a7b6b}</style>
<g filter="url(#roughG)" fill="none" stroke-linecap="round" stroke-linejoin="round">
<path d="M 204 48 C 215 52, 235 92, 262 112 M 204 98 C 205 100, 232 112, 258 120 M 204 148 C 205 146, 232 136, 258 130 M 204 198 C 215 194, 235 158, 262 138" stroke="#b5a594" stroke-width="1.4"/>
<ellipse cx="318" cy="125" rx="54" ry="26" stroke="#4e4338" stroke-width="1.7" fill="#f3ebe0"/>
<path d="M 376 125 H 420" stroke="#9a7048" stroke-width="1.8"/>
<path d="M 410 117 L 421 125 L 410 133" stroke="#9a7048" stroke-width="1.8"/>
<path d="M 432 180 H 632" stroke="#b5a594" stroke-width="1.3"/>
<path d="M 436 178 C 460 176, 470 96, 496 94 C 522 96, 532 176, 560 178" stroke="#8a7b6b" stroke-width="1.5" stroke-dasharray="5 5"/>
<path d="M 496 178 C 524 176, 536 70, 566 68 C 596 70, 606 176, 630 178" stroke="#4e4338" stroke-width="1.9"/>
<path d="M 506 52 H 548" stroke="#9a7048" stroke-width="1.6"/>
<path d="M 539 45 L 549 52 L 539 59" stroke="#9a7048" stroke-width="1.6"/>
</g>
<text x="198" y="54" class="hw" font-size="19" text-anchor="end">better reasoning</text>
<text x="198" y="104" class="hw" font-size="19" text-anchor="end">better answers</text>
<text x="198" y="154" class="hw" font-size="19" text-anchor="end">the right format</text>
<text x="198" y="204" class="hw" font-size="19" text-anchor="end">following instructions</text>
<text x="318" y="132" class="hw" font-size="22" text-anchor="middle">reward</text>
<text x="470" y="200" class="hw-s" font-size="17" text-anchor="middle">pretrained</text>
<text x="580" y="200" class="hw-s" font-size="17" text-anchor="middle" style="fill:#4e4338">post-trained</text>
<text x="532" y="228" class="hw-s" font-size="17" text-anchor="middle">distribution over behaviors</text>
</svg>
<figcaption>Four goals on the left, and one kind of change on the right: the pretrained distribution gets moved.</figcaption>
</figure>

So the question in the title is meant fairly literally:

> Can everything RL post-training does be understood as taking the pretrained model's distribution and shifting it toward the outputs people want?

I think the answer is mostly yes, and the math behind it fits in a few lines.

---

## One distribution, scored in different ways

For a prompt $x$, the model defines a distribution over responses $y$:

$$\pi_\theta(y \mid x).$$

A single response is rarely what we care about. We care about kinds of responses, such as the correct ones or the ones in valid JSON. A kind of response is a set $B_x$, which depends on the prompt because the right answer and the constraints do. The probability that the model behaves that way is the total mass inside the set:

$$\pi_\theta(B_x \mid x) = \sum_{y \in B_x} \pi_\theta(y \mid x).$$

Many of the numbers we report about a model have this form. If responses are sampled directly from the policy, accuracy is

$$\text{Accuracy} = \mathbb{E}_{x \sim p_{\text{eval}}}\big[\pi_\theta(B_x \mid x)\big],$$

with $B_x$ the set of correct responses. Format compliance is the same expression with a different set. Graded measures like quality don't need to be forced into a set. They are more naturally an expected score:

$$\text{Quality} = \mathbb{E}_{x \sim p_{\text{eval}}}\; \mathbb{E}_{y \sim \pi_\theta(\cdot \mid x)}\big[s(x, y)\big].$$

One condition applies. Greedy decoding, temperature and top-p all change which responses actually come out, so these identities describe sampling accuracy. Any comparison between two models has to keep the decoding setup fixed.

With that in place, accuracy, format compliance and quality are different ways of scoring the same conditional distribution. Improving any of them means changing that distribution.

A small made-up example makes this concrete, and it comes back later. Suppose the prompt is a math question that ends with "reply in JSON only." Every response is either right or wrong, and either valid JSON or not, so there are four kinds. Say the model starts out like this:

<figure class="post-figure post-figure--doodle">
<svg viewBox="0 0 660 240" xmlns="http://www.w3.org/2000/svg" role="img" aria-label="A doodle of four jars holding different amounts of water, with an arrow pointing at the emptiest jar, the one we want fuller. A note says the jars always add up to one hundred percent.">
<defs>
<filter id="roughH" x="-5%" y="-5%" width="110%" height="110%"><feTurbulence type="fractalNoise" baseFrequency="0.03" numOctaves="2" seed="23" result="n"/><feDisplacementMap in="SourceGraphic" in2="n" scale="2.2" xChannelSelector="R" yChannelSelector="G"/></filter>
</defs>
<style>.hw{font-family:'Caveat','Cormorant Garamond',cursive;fill:#4e4338}.hw-s{font-family:'Caveat','Cormorant Garamond',cursive;fill:#8a7b6b}</style>
<g filter="url(#roughH)" fill="none" stroke-linecap="round" stroke-linejoin="round">
<path d="M 92 168 H 148 V 180 H 92 Z" fill="#e6e9dc" stroke="none"/>
<path d="M 222 138 H 278 V 180 H 222 Z" fill="#ecdfcd" stroke="none"/>
<path d="M 352 150 H 408 V 180 H 352 Z" fill="#ecdfcd" stroke="none"/>
<path d="M 482 144 H 538 V 180 H 482 Z" fill="#f0dcd4" stroke="none"/>
<path d="M 90 70 V 182 H 150 V 70 M 220 70 V 182 H 280 V 70 M 350 70 V 182 H 410 V 70 M 480 70 V 182 H 540 V 70" stroke="#4e4338" stroke-width="1.7"/>
<path d="M 92 168 H 148" stroke="#6b7a52" stroke-width="1.5"/>
<path d="M 222 138 H 278 M 352 150 H 408" stroke="#9a7048" stroke-width="1.5"/>
<path d="M 482 144 H 538" stroke="#a8543f" stroke-width="1.5"/>
<path d="M 506 60 C 440 14, 200 14, 128 58" stroke="#9a7048" stroke-width="1.8" stroke-dasharray="6 5"/>
<path d="M 142 44 L 126 59 L 146 62" stroke="#9a7048" stroke-width="1.8"/>
</g>
<text x="120" y="204" class="hw" font-size="18" text-anchor="middle">right, in JSON</text>
<text x="250" y="204" class="hw" font-size="18" text-anchor="middle">right, not JSON</text>
<text x="380" y="204" class="hw" font-size="18" text-anchor="middle">wrong, in JSON</text>
<text x="510" y="204" class="hw" font-size="18" text-anchor="middle">wrong, not JSON</text>
<text x="120" y="160" class="hw-s" font-size="16" text-anchor="middle">10%</text>
<text x="250" y="130" class="hw-s" font-size="16" text-anchor="middle">35%</text>
<text x="380" y="142" class="hw-s" font-size="16" text-anchor="middle">25%</text>
<text x="510" y="136" class="hw-s" font-size="16" text-anchor="middle">30%</text>
<text x="318" y="52" class="hw" font-size="18" text-anchor="middle" style="fill:#9a7048">the goal is to move more of it here</text>
<text x="600" y="110" class="hw-s" font-size="17" text-anchor="middle">always</text>
<text x="600" y="130" class="hw-s" font-size="17" text-anchor="middle">100%</text>
<text x="600" y="150" class="hw-s" font-size="17" text-anchor="middle">in total</text>
</svg>
<figcaption>An illustrative starting point: one prompt and four kinds of response.</figcaption>
</figure>

The model is correct 45% of the time and returns valid JSON 35% of the time. What we want is the first jar, which holds only 10%.

---

## What a reward does to the distribution

To move mass into that first jar, RL needs an objective. The standard one is expected reward:

$$J(\theta) = \mathbb{E}_{x \sim p(x)}\; \mathbb{E}_{y \sim \pi_\theta(\cdot \mid x)} \big[R(x, y)\big].$$

If the reward is 1 for responses in $B_x$ and 0 otherwise, the inner expectation is exactly $\pi_\theta(B_x \mid x)$. In that case, maximizing expected reward and putting more mass in the set are the same statement.

There is an idealized version of this problem that can be solved exactly, and it shows how a reward tilts a distribution. Fix a prompt and a reward function, and add a penalty for moving away from a reference policy $\pi_{\text{ref}}$:

$$\max_{\pi} \;\; \mathbb{E}_{y \sim \pi}\big[R(y)\big] \;-\; \beta\, D_{\mathrm{KL}}\big(\pi \,\Vert\, \pi_{\text{ref}}\big), \qquad \beta > 0.$$

Suppose $\pi$ can be any distribution over the responses that $\pi_{\text{ref}}$ supports, and the normalizing constant $Z$ below is finite. Then the solution is

$$\pi^{\star}(y) = \frac{1}{Z}\, \pi_{\text{ref}}(y)\, \exp\!\left(\frac{R(y)}{\beta}\right).$$

The optimal policy is the reference policy with each response scaled by an exponential of its reward. If the reference is the model we start from, then the new model is the old one with its probabilities tilted toward what the reward prefers. Written as log-odds between two responses, it is even plainer:

$$\log \frac{\pi^{\star}(y)}{\pi^{\star}(y')} = \log \frac{\pi_{\text{ref}}(y)}{\pi_{\text{ref}}(y')} + \frac{R(y) - R(y')}{\beta}.$$

The reward difference is added to the log-odds, and $\beta$ decides how much it counts. A large $\beta$ keeps the policy close to where it started, and a small one lets the reward take over.

This objective is close to what LLM post-training actually uses. The RL stage of InstructGPT (Ouyang et al., 2022) adds a per-token KL penalty from the supervised model, so that the policy doesn't over-optimize the reward model. The DPO paper (Rafailov et al., 2023) states the same solution as its Equation 4 and derives it in Appendix A.1.

The solution should still be read as a reference point. It describes one prompt at a time and a policy that is free to be any distribution. Real training has shared weights, a limited number of samples and a particular algorithm, and nothing guarantees that it arrives at $\pi^{\star}$. What the formula gives is a clear picture of the direction in which a reward pushes.

---

## Different rewards, the same mechanism

In practice the reward usually has several parts. Schematically:

$$R(x,y) = w_1 R_{\text{accuracy}} + w_2 R_{\text{quality}} + w_3 R_{\text{format}} + w_4 R_{\text{instruction}}.$$

Real systems don't always combine them as a simple weighted sum, and the parts come from different places. Accuracy and format can often be checked by a rule, which is the setting usually called RL with verifiable rewards, or RLVR. This is how DeepSeek-R1-Zero was trained to reason. Its reward has an accuracy part, which checks the final answer, and a format part, which checks that the thinking sits inside the expected tags. Quality usually needs human preferences, or a reward model trained on them, which is the RLHF setup of InstructGPT.

Whatever the source, each part marks some responses as preferred, and the optimization pushes mass toward them.

| What we say improves | What the reward moves |
| --- | --- |
| Reasoning accuracy | more mass on responses that reach the correct answer |
| Response quality | more mass on responses the rater or reward model prefers |
| Formatting | more mass on outputs that meet the format |
| Instruction following | more mass on responses that satisfy the stated constraints |
| Safety and alignment | mass moved toward responses that fit the behavior guidelines |

The two columns are not quite the same thing. A reward on the final answer moves mass toward responses that end correctly, and that doesn't verify the reasoning written before the answer. A quality reward moves mass toward what the scorer likes. A reward is a proxy for the behavior we want, so succeeding at the optimization doesn't automatically mean the behavior itself got better.

The closed-form solution shows how the parts fit together. With a sum of rewards in the exponent, the optimal policy is a product:

$$\pi^{\star}(y) \;\propto\; \pi_{\text{ref}}(y)\; \exp\!\left(\frac{w_1 R_{\text{accuracy}}(y)}{\beta}\right) \exp\!\left(\frac{w_3 R_{\text{format}}(y)}{\beta}\right) \cdots$$

Each objective contributes its own factor, and all the factors act on the same distribution.

The four jars show what this looks like with numbers. Take the starting policy from before, give a reward of 1 for a correct answer and 1 for valid JSON, and set $\beta = 0.5$. The formula gives the optimal policy for each choice of reward.

<figure class="post-figure">
<svg viewBox="0 0 660 262" xmlns="http://www.w3.org/2000/svg" role="img" aria-label="Probability mass over four kinds of response, before training and under three different rewards">
<style>.lb{font-family:'Source Sans 3',system-ui,sans-serif;font-size:11px;fill:#8a7b6b}.lbs{font-family:'Source Sans 3',system-ui,sans-serif;font-size:11.5px;fill:#3d342b}.mono{font-family:ui-monospace,monospace;font-size:10px;fill:#6f6254}</style>
<text x="18" y="22" class="lbs" font-weight="600">Where the probability sits, under four rewards</text>
<text x="114" y="40" class="lbs" text-anchor="middle">before</text>
<text x="85.0" y="56" class="lb" text-anchor="middle">JSON</text><text x="143.0" y="56" class="lb" text-anchor="middle">not JSON</text>
<rect x="56" y="62" width="58" height="58" fill="none" stroke="#e4d5c4" stroke-width="1"/>
<rect x="74.8" y="80.8" width="20.4" height="20.4" rx="2" fill="#e6e9dc" stroke="#6b7a52" stroke-width="1"/>
<text x="85.0" y="94.5" class="mono" text-anchor="middle">10%</text>
<rect x="114" y="62" width="58" height="58" fill="none" stroke="#e4d5c4" stroke-width="1"/>
<rect x="123.9" y="71.9" width="38.2" height="38.2" rx="2" fill="#ecdfcd" stroke="#9a7048" stroke-width="1"/>
<text x="143.0" y="94.5" class="mono" text-anchor="middle">35%</text>
<rect x="56" y="120" width="58" height="58" fill="none" stroke="#e4d5c4" stroke-width="1"/>
<rect x="68.9" y="132.9" width="32.3" height="32.3" rx="2" fill="#ecdfcd" stroke="#9a7048" stroke-width="1"/>
<text x="85.0" y="152.5" class="mono" text-anchor="middle">25%</text>
<rect x="114" y="120" width="58" height="58" fill="none" stroke="#e4d5c4" stroke-width="1"/>
<rect x="125.3" y="131.3" width="35.4" height="35.4" rx="2" fill="#f0dcd4" stroke="#a8543f" stroke-width="1"/>
<text x="143.0" y="152.5" class="mono" text-anchor="middle">30%</text>
<text x="114" y="198" class="lb" text-anchor="middle">correct 45%</text>
<text x="114" y="213" class="lb" text-anchor="middle">valid JSON 35%</text>
<text x="266" y="40" class="lbs" text-anchor="middle">reward accuracy</text>
<text x="237.0" y="56" class="lb" text-anchor="middle">JSON</text><text x="295.0" y="56" class="lb" text-anchor="middle">not JSON</text>
<rect x="208" y="62" width="58" height="58" fill="none" stroke="#e4d5c4" stroke-width="1"/>
<rect x="222.9" y="76.9" width="28.2" height="28.2" rx="2" fill="#e6e9dc" stroke="#6b7a52" stroke-width="1"/>
<text x="237.0" y="94.5" class="mono" text-anchor="middle">19%</text>
<rect x="266" y="62" width="58" height="58" fill="none" stroke="#e4d5c4" stroke-width="1"/>
<rect x="268.6" y="64.6" width="52.7" height="52.7" rx="2" fill="#ecdfcd" stroke="#9a7048" stroke-width="1"/>
<text x="295.0" y="94.5" class="mono" text-anchor="middle">67%</text>
<rect x="208" y="120" width="58" height="58" fill="none" stroke="#e4d5c4" stroke-width="1"/>
<rect x="228.8" y="140.8" width="16.4" height="16.4" rx="2" fill="#ecdfcd" stroke="#9a7048" stroke-width="1"/>
<text x="237.0" y="152.5" class="mono" text-anchor="middle">6%</text>
<rect x="266" y="120" width="58" height="58" fill="none" stroke="#e4d5c4" stroke-width="1"/>
<rect x="286.0" y="140.0" width="18.0" height="18.0" rx="2" fill="#f0dcd4" stroke="#a8543f" stroke-width="1"/>
<text x="295.0" y="152.5" class="mono" text-anchor="middle">8%</text>
<text x="266" y="198" class="lb" text-anchor="middle">correct 86%</text>
<text x="266" y="213" class="lb" text-anchor="middle">valid JSON 26%</text>
<text x="418" y="40" class="lbs" text-anchor="middle">reward the format</text>
<text x="389.0" y="56" class="lb" text-anchor="middle">JSON</text><text x="447.0" y="56" class="lb" text-anchor="middle">not JSON</text>
<rect x="360" y="62" width="58" height="58" fill="none" stroke="#e4d5c4" stroke-width="1"/>
<rect x="373.6" y="75.6" width="30.8" height="30.8" rx="2" fill="#e6e9dc" stroke="#6b7a52" stroke-width="1"/>
<text x="389.0" y="94.5" class="mono" text-anchor="middle">23%</text>
<rect x="418" y="62" width="58" height="58" fill="none" stroke="#e4d5c4" stroke-width="1"/>
<rect x="436.4" y="80.4" width="21.2" height="21.2" rx="2" fill="#ecdfcd" stroke="#9a7048" stroke-width="1"/>
<text x="447.0" y="94.5" class="mono" text-anchor="middle">11%</text>
<rect x="360" y="120" width="58" height="58" fill="none" stroke="#e4d5c4" stroke-width="1"/>
<rect x="364.6" y="124.6" width="48.8" height="48.8" rx="2" fill="#ecdfcd" stroke="#9a7048" stroke-width="1"/>
<text x="389.0" y="152.5" class="mono" text-anchor="middle">57%</text>
<rect x="418" y="120" width="58" height="58" fill="none" stroke="#e4d5c4" stroke-width="1"/>
<rect x="437.2" y="139.2" width="19.7" height="19.7" rx="2" fill="#f0dcd4" stroke="#a8543f" stroke-width="1"/>
<text x="447.0" y="152.5" class="mono" text-anchor="middle">9%</text>
<text x="418" y="198" class="lb" text-anchor="middle">correct 34%</text>
<text x="418" y="213" class="lb" text-anchor="middle">valid JSON 80%</text>
<text x="570" y="40" class="lbs" text-anchor="middle">reward both</text>
<text x="541.0" y="56" class="lb" text-anchor="middle">JSON</text><text x="599.0" y="56" class="lb" text-anchor="middle">not JSON</text>
<rect x="512" y="62" width="58" height="58" fill="none" stroke="#e4d5c4" stroke-width="1"/>
<rect x="517.4" y="67.4" width="47.2" height="47.2" rx="2" fill="#e6e9dc" stroke="#6b7a52" stroke-width="1"/>
<text x="541.0" y="94.5" class="mono" text-anchor="middle">54%</text>
<rect x="570" y="62" width="58" height="58" fill="none" stroke="#e4d5c4" stroke-width="1"/>
<rect x="582.7" y="74.7" width="32.5" height="32.5" rx="2" fill="#ecdfcd" stroke="#9a7048" stroke-width="1"/>
<text x="599.0" y="94.5" class="mono" text-anchor="middle">25%</text>
<rect x="512" y="120" width="58" height="58" fill="none" stroke="#e4d5c4" stroke-width="1"/>
<rect x="527.3" y="135.3" width="27.5" height="27.5" rx="2" fill="#ecdfcd" stroke="#9a7048" stroke-width="1"/>
<text x="541.0" y="152.5" class="mono" text-anchor="middle">18%</text>
<rect x="570" y="120" width="58" height="58" fill="none" stroke="#e4d5c4" stroke-width="1"/>
<rect x="593.5" y="143.5" width="11.1" height="11.1" rx="2" fill="#f0dcd4" stroke="#a8543f" stroke-width="1"/>
<text x="599.0" y="152.5" class="mono" text-anchor="middle">3%</text>
<text x="570" y="198" class="lb" text-anchor="middle">correct 79%</text>
<text x="570" y="213" class="lb" text-anchor="middle">valid JSON 72%</text>
<text x="48" y="95.0" class="lb" text-anchor="end">right</text><text x="48" y="153.0" class="lb" text-anchor="end">wrong</text>
<text x="18" y="252" class="lb">Square area is proportional to probability. Each panel sums to 100%.</text>
</svg>
<figcaption>The same illustrative starting policy under three rewards, computed from the closed-form solution with β = 0.5.</figcaption>
</figure>

Rewarding accuracy alone takes the model from 45% to 86% correct. It also lowers format compliance from 35% to 26%, although nothing in the reward mentions format. Rewarding the format alone does the reverse: valid JSON goes from 35% to 80%, and accuracy falls from 45% to 34%. Rewarding both moves more than half of the mass into the jar we wanted, and both numbers improve, to 79% and 72%.

So rewarding one thing hurt the other here, and the fixed total of probability is not enough to explain it. A fixed total only means that when one set gains mass, its complement loses the same amount. It doesn't say which responses lose, and the two sets overlap, so both could have grown together.

The reason is in the starting distribution. Write $A$ for the correct responses and $F$ for the ones in valid JSON. If the two were unrelated, 45% × 35% ≈ 16% of the mass would sit in both. Only 10% does:

$$\operatorname{Cov}(\mathbf{1}_A, \mathbf{1}_F) = 0.10 - 0.45 \times 0.35 = -0.0575.$$

In this starting policy, correct responses are less likely than average to be valid JSON. So tilting toward correctness also tilts away from JSON. If the covariance were zero, rewarding accuracy would leave format compliance where it was. If it were positive, format compliance would rise as well, with no reward on it at all.[^cov]

So a reward on one objective can raise, lower, or leave alone a metric it never mentions, depending on how the two are related in the current distribution. The unified view is about how the improvements happen. It makes no promise that they all happen at once.

---

## What real training adds

The closed form skips over how the mass actually gets moved. In practice the model samples responses, they get scored, and the scores turn into a gradient:

$$\nabla_\theta J = \mathbb{E}\Big[\big(R(x,y) - b(x)\big)\, \nabla_\theta \log \pi_\theta(y \mid x)\Big].$$

Here $b(x)$ is a baseline, such as the average reward for that prompt. It can depend on the prompt but not on the sampled response, which is what keeps the gradient unbiased. A sample that scores above the baseline contributes an update in the direction that raises its log-probability, and a sample below it contributes the opposite. The actual change in any one response's probability depends on all the samples together, on the shared weights, and on any regularization.

<figure class="post-figure post-figure--doodle">
<svg viewBox="0 0 660 255" xmlns="http://www.w3.org/2000/svg" role="img" aria-label="A doodle of a loop with three bubbles: policy, samples and reward. The policy tries things, the samples get scored, and the reward updates the policy. A note in the middle says that what is never tried is never scored.">
<defs>
<filter id="roughE" x="-5%" y="-5%" width="110%" height="110%"><feTurbulence type="fractalNoise" baseFrequency="0.03" numOctaves="2" seed="17" result="n"/><feDisplacementMap in="SourceGraphic" in2="n" scale="2.2" xChannelSelector="R" yChannelSelector="G"/></filter>
</defs>
<style>.hw{font-family:'Caveat','Cormorant Garamond',cursive;fill:#4e4338}.hw-s{font-family:'Caveat','Cormorant Garamond',cursive;fill:#8a7b6b}</style>
<g filter="url(#roughE)" fill="none" stroke-linecap="round" stroke-linejoin="round">
<ellipse cx="330" cy="52" rx="62" ry="26" stroke="#4e4338" stroke-width="1.7" fill="#f3ebe0"/>
<ellipse cx="482" cy="188" rx="62" ry="26" stroke="#4e4338" stroke-width="1.7" fill="#f3ebe0"/>
<ellipse cx="178" cy="188" rx="62" ry="26" stroke="#4e4338" stroke-width="1.7" fill="#f3ebe0"/>
<path d="M 394 62 C 440 78, 470 112, 478 156" stroke="#9a7048" stroke-width="1.8"/>
<path d="M 468 146 L 478 157 L 484 144" stroke="#9a7048" stroke-width="1.8"/>
<path d="M 424 202 C 376 224, 286 224, 238 202" stroke="#9a7048" stroke-width="1.8"/>
<path d="M 252 199 L 237 201 L 246 213" stroke="#9a7048" stroke-width="1.8"/>
<path d="M 182 156 C 190 112, 220 78, 266 62" stroke="#9a7048" stroke-width="1.8"/>
<path d="M 253 58 L 267 62 L 258 73" stroke="#9a7048" stroke-width="1.8"/>
</g>
<text x="330" y="59" class="hw" font-size="22" text-anchor="middle">policy</text>
<text x="482" y="195" class="hw" font-size="22" text-anchor="middle">samples</text>
<text x="178" y="195" class="hw" font-size="22" text-anchor="middle">reward</text>
<text x="472" y="98" class="hw-s" font-size="17">tries a few things</text>
<text x="330" y="244" class="hw-s" font-size="17" text-anchor="middle">they get scored</text>
<text x="190" y="98" class="hw-s" font-size="17" text-anchor="end">nudges the policy</text>
<text x="330" y="138" class="hw" font-size="18" text-anchor="middle" style="fill:#a8543f">what it never tries</text>
<text x="330" y="158" class="hw" font-size="18" text-anchor="middle" style="fill:#a8543f">never gets scored</text>
</svg>
<figcaption>The loop that on-policy training runs in.</figcaption>
</figure>

Two things follow that the closed form doesn't show. Responses the policy rarely produces get fewer chances to be reinforced directly, so where training ends up depends on where it starts. And because all prompts share one set of weights, an update made for one prompt also moves the distribution on others.

---

## RL post-training as distribution control

Pretraining gives a model a distribution that reflects the text it has read. Post-training moves that distribution toward what people want from it. We tend to sort the results into separate abilities: better reasoning, higher quality, closer instruction following, more reliable formatting. From the side of the optimization they are not separate. The reward, whether it comes from human preferences or from a rule-based check, defines which behaviors we want, and training changes how likely the model is to produce them. In that sense RL is a game of distributions.

This view describes what changed in the model's behavior under a given reward. The reward accounts for the direction of the move, and the starting distribution accounts for what came along with it. Better reasoning, higher quality and cleaner formatting are then different measurements of that one change.

What I find useful is that it turns questions about abilities into questions about probability mass. When a metric moves after RL in a way nobody intended, the first thing I would check is how that metric was related to the reward in the distribution the model started from.

---

## Notes

References:

- Rafailov, Sharma, Mitchell, Ermon, Manning & Finn, *Direct Preference Optimization: Your Language Model is Secretly a Reward Model*, 2023 — [arXiv:2305.18290](https://arxiv.org/abs/2305.18290). Equation 4 and Appendix A.1 give the closed-form solution used here.
- Ouyang et al., *Training Language Models to Follow Instructions with Human Feedback*, 2022 — [arXiv:2203.02155](https://arxiv.org/abs/2203.02155)
- DeepSeek-AI, *DeepSeek-R1: Incentivizing Reasoning Capability in LLMs via Reinforcement Learning*, Nature 2025 — [arXiv:2501.12948](https://arxiv.org/abs/2501.12948). The accuracy and format rewards are described in the section on DeepSeek-R1-Zero.
- Levine, *Reinforcement Learning and Control as Probabilistic Inference: Tutorial and Review*, 2018 — [arXiv:1805.00909](https://arxiv.org/abs/1805.00909). Background on the same idealized objective.

The four-jar numbers are made up for illustration. The percentages in the last figure are computed from them with the closed-form solution, using a reward of 1 per satisfied objective and β = 0.5.

[^cov]: The general statement: for the family $$\pi_\lambda(y) \propto \pi_{\text{ref}}(y)\, e^{\lambda R(y)}$$ and any metric $$f$$, we have $$\frac{d}{d\lambda}\, \mathbb{E}_{\pi_\lambda}[f] = \operatorname{Cov}_{\pi_\lambda}(f, R)$$. Turning up a reward moves another metric at a rate equal to their covariance under the current policy, which can be positive, negative or zero.

---

{% include citation.html
  title="Is Reinforcement Learning Fundamentally a Game of Distributions?"
  author="Renee Jia"
  journal="renee-jia.github.io"
  year="2026"
  url="https://renee-jia.github.io/research%20blog/ai%20research/is-reinforcement-learning-fundamentally-a-game-of-distributions/"
  bibtex_key="reneejia2026gameofdistributions"
%}
