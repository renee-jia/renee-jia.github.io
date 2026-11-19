---
title: "How Do We Learn to Think? Reflections on Training AI Learners"
date: 2026-11-18
research_kind: perspective
categories:
  - Research Blog
  - AI Research
  - Large Language Models
tags:
  - training
  - memory
  - reasoning
  - continual learning
  - AI philosophy
excerpt: "What if a person had 10,000× more memory? How would she learn? And how much do we really know about teaching a learner like that?"
read_time: "5 min read"
layout: distill
last_modified_at: 2026-11-18T08:00:00-08:00
---

Lately, I've been thinking about how we train AI models, and somehow it keeps reminding me of how humans learn.

When I think about my own learning experience, it seems to start with remembering things, then imitating others, and eventually learning through trial and error.

Interestingly, isn't that somewhat similar to how we train LLMs today?

I sometimes imagine pretraining as educating a college student who has studied almost every subject imaginable: mathematics, physics, computer science, history, philosophy, and so on. She has acquired an enormous amount of knowledge about the world, but hasn't necessarily chosen a major yet.

Then comes learning from domain experts. Professors, mentors and senior colleagues show her how problems in a field are actually approached, and at first she mostly follows their examples.

After that she tries out real jobs. That is where she learns to solve real-world problems, which are rarely as clean as the ones in a textbook. She makes mistakes, gets feedback, and gradually figures out what works.

Of course, human learning is much messier than these neatly separated stages. But the resemblance is interesting.

## What if a person had 10,000× more memory?

When I was a kid, I was never particularly good at memorization. I remember struggling to recite passages from textbooks, and I used to envy those classmates who seemed to remember everything after reading it just once.

Especially in subjects like literature and history, having a great memory seemed like such an advantage. If you could effortlessly remember everything you'd learned, wouldn't learning become so much easier?

Now, seeing AI demonstrate remarkable, and sometimes superhuman, abilities across so many tasks, I find myself revisiting that childhood curiosity.

Imagine someone who could remember almost every book she's read, every problem she's solved, every experiment she's attempted, and every mistake she's made.

<figure class="post-figure post-figure--doodle">
<svg viewBox="0 0 660 285" xmlns="http://www.w3.org/2000/svg" role="img" aria-label="A doodle of two stick figures. The left one stands by a small bookshelf with a few books. The right one stands by a tall bookshelf packed with books, with a thought asking whether she thinks differently or just looks more up.">
<defs>
<filter id="roughA" x="-5%" y="-5%" width="110%" height="110%"><feTurbulence type="fractalNoise" baseFrequency="0.03" numOctaves="2" seed="5" result="n"/><feDisplacementMap in="SourceGraphic" in2="n" scale="2.2" xChannelSelector="R" yChannelSelector="G"/></filter>
</defs>
<style>.hw{font-family:'Caveat','Cormorant Garamond',cursive;fill:#4e4338}.hw-s{font-family:'Caveat','Cormorant Garamond',cursive;fill:#8a7b6b}</style>
<g filter="url(#roughA)" fill="none" stroke-linecap="round" stroke-linejoin="round">
<path d="M 40 250 H 620" stroke="#b5a594" stroke-width="1.2"/>
<!-- small shelf -->
<path d="M 130 250 V 150 H 210 V 250 M 130 183 H 210 M 130 216 H 210" stroke="#4e4338" stroke-width="1.6"/>
<path d="M 140 182 V 164 M 149 182 V 161 M 158 182 V 166 M 146 215 V 197 M 172 249 V 231 M 181 249 V 228 M 190 249 V 233" stroke="#4e4338" stroke-width="3.2"/>
<path d="M 155 215 V 195 M 163 249 V 230" stroke="#9a7048" stroke-width="3.2"/>
<!-- left figure -->
<path d="M 90 189 a 7 7 0 1 0 0.1 0 M 90 203 V 230 M 90 212 L 80 222 M 90 212 L 102 218 M 90 230 L 83 250 M 90 230 L 97 250" stroke="#4e4338" stroke-width="1.5"/>
<!-- tall shelf -->
<path d="M 430 250 V 30 H 560 V 250 M 430 58 H 560 M 430 85 H 560 M 430 113 H 560 M 430 140 H 560 M 430 168 H 560 M 430 195 H 560 M 430 223 H 560" stroke="#4e4338" stroke-width="1.6"/>
<path d="M 439 57 V 40 M 448 57 V 37 M 464 57 V 38 M 473 57 V 36 M 481 57 V 41 M 489 57 V 39 M 506 57 V 37 M 514 57 V 42 M 523 57 V 38 M 531 57 V 36 M 548 57 V 39 M 439 84 V 67 M 447 84 V 64 M 456 84 V 69 M 472 84 V 63 M 481 84 V 68 M 489 84 V 66 M 497 84 V 67 M 514 84 V 69 M 522 84 V 65 M 531 84 V 63 M 539 84 V 68 M 439 112 V 95 M 447 112 V 92 M 455 112 V 97 M 464 112 V 93 M 480 112 V 96 M 489 112 V 94 M 497 112 V 95 M 505 112 V 92 M 522 112 V 93 M 530 112 V 91 M 539 112 V 96 M 547 112 V 94 M 448 139 V 119 M 456 139 V 124 M 464 139 V 120 M 473 139 V 118 M 489 139 V 121 M 498 139 V 122 M 506 139 V 119 M 514 139 V 124 M 531 139 V 118 M 539 139 V 123 M 548 139 V 121 M 439 167 V 150 M 456 167 V 152 M 464 167 V 148 M 472 167 V 146 M 481 167 V 151 M 497 167 V 150 M 506 167 V 147 M 514 167 V 152 M 522 167 V 148 M 539 167 V 151 M 547 167 V 149 M 439 194 V 177 M 447 194 V 174 M 464 194 V 175 M 472 194 V 173 M 480 194 V 178 M 489 194 V 176 M 505 194 V 174 M 514 194 V 179 M 522 194 V 175 M 530 194 V 173 M 547 194 V 176 M 439 222 V 205 M 448 222 V 202 M 456 222 V 207 M 473 222 V 201 M 481 222 V 206 M 489 222 V 204 M 498 222 V 205 M 514 222 V 207 M 523 222 V 203 M 531 222 V 201 M 539 222 V 206 M 439 249 V 232 M 447 249 V 229 M 456 249 V 234 M 464 249 V 230 M 481 249 V 233 M 489 249 V 231 M 497 249 V 232 M 506 249 V 229 M 522 249 V 230 M 531 249 V 228 M 539 249 V 233 M 547 249 V 231" stroke="#4e4338" stroke-width="3.2"/>
<path d="M 456 57 V 42 M 498 57 V 40 M 539 57 V 41 M 464 84 V 65 M 506 84 V 64 M 547 84 V 66 M 472 112 V 91 M 514 112 V 97 M 439 139 V 122 M 481 139 V 123 M 523 139 V 120 M 447 167 V 147 M 489 167 V 149 M 531 167 V 146 M 455 194 V 179 M 497 194 V 177 M 539 194 V 178 M 464 222 V 203 M 506 222 V 202 M 548 222 V 204 M 472 249 V 228 M 514 249 V 234" stroke="#9a7048" stroke-width="3.2"/>
<!-- right figure -->
<path d="M 390 189 a 7 7 0 1 0 0.1 0 M 390 203 V 230 M 390 212 L 378 218 M 390 212 L 402 204 M 390 230 L 383 250 M 390 230 L 397 250" stroke="#4e4338" stroke-width="1.5"/>
<!-- thought dots -->
<path d="M 376 176 a 2.5 2.5 0 1 0 0.1 0 M 366 160 a 3.5 3.5 0 1 0 0.1 0" stroke="#8a7b6b" stroke-width="1.2"/>
</g>
<text x="170" y="274" class="hw" font-size="18" text-anchor="middle">what I can hold</text>
<text x="495" y="274" class="hw" font-size="18" text-anchor="middle" style="fill:#9a7048">10,000× that</text>
<text x="305" y="118" class="hw-s" font-size="18" text-anchor="middle">does she think differently,</text>
<text x="305" y="140" class="hw-s" font-size="18" text-anchor="middle">or just look more up?</text>
</svg>
<figcaption>The same person, with a much bigger shelf.</figcaption>
</figure>

How would such a person learn? How would she develop expertise? Would she reason differently, or simply have far more knowledge to reason with?

And could some of the impressive capabilities we see in AI today be understood through a similar lens: an intelligence that has absorbed more information and patterns than any individual human could?

The more I think about this hypothetical person, the more intuitive our current AI training pipeline starts to feel.

- Pretraining. Absorbing an enormous amount of information, almost like reading and reciting everything, except that the process also develops abstractions and connections between concepts.
- Mid-training. Going deeper into a particular domain, perhaps like choosing a major and developing specialized knowledge.
- Post-training (SFT and RL). Learning how to apply that knowledge, first through demonstrations and imitation, then through practice and feedback.
- Continual learning? Perhaps something closer to learning and evolving at the same time, where each new experience has the potential to shape how we approach the next.

<figure class="post-figure post-figure--doodle">
<svg viewBox="0 0 660 225" xmlns="http://www.w3.org/2000/svg" role="img" aria-label="A doodle of a path with four stops: a stack of books labeled read everything, a signpost labeled pick a major, a marked worksheet labeled watch then practice, and a dashed looping arrow labeled keep learning with a question mark.">
<defs>
<filter id="roughB" x="-5%" y="-5%" width="110%" height="110%"><feTurbulence type="fractalNoise" baseFrequency="0.03" numOctaves="2" seed="14" result="n"/><feDisplacementMap in="SourceGraphic" in2="n" scale="2.2" xChannelSelector="R" yChannelSelector="G"/></filter>
</defs>
<style>.hw{font-family:'Caveat','Cormorant Garamond',cursive;fill:#4e4338}.hw-s{font-family:'Caveat','Cormorant Garamond',cursive;fill:#8a7b6b}</style>
<g filter="url(#roughB)" fill="none" stroke-linecap="round" stroke-linejoin="round">
<!-- the path -->
<path d="M 40 152 C 120 140, 180 162, 250 150 C 320 138, 350 162, 410 150 C 440 144, 465 150, 490 150" stroke="#4e4338" stroke-width="1.8"/>
<path d="M 490 150 C 520 150, 550 146, 615 150" stroke="#9a7048" stroke-width="1.8" stroke-dasharray="5 5"/>
<path d="M 90 144 a 5 5 0 1 0 0.1 0 M 250 145 a 5 5 0 1 0 0.1 0 M 410 145 a 5 5 0 1 0 0.1 0" stroke="#4e4338" stroke-width="1.6" fill="#f3ebe0"/>
<path d="M 570 144 a 5 5 0 1 0 0.1 0" stroke="#9a7048" stroke-width="1.6" fill="#f3ebe0"/>
<!-- stack of books -->
<path d="M 60 106 H 120 V 118 H 60 Z M 66 93 H 114 V 105 H 66 Z M 62 80 H 118 V 92 H 62 Z M 70 67 H 108 V 79 H 70 Z" stroke="#4e4338" stroke-width="1.5"/>
<path d="M 70 106 V 118 M 76 93 V 105 M 72 80 V 92 M 80 67 V 79" stroke="#9a7048" stroke-width="1.5"/>
<!-- signpost -->
<path d="M 250 128 V 62" stroke="#4e4338" stroke-width="1.6"/>
<path d="M 250 68 H 284 L 293 77 L 284 86 H 250 Z" stroke="#6b7a52" stroke-width="1.6"/>
<path d="M 250 94 H 220 L 212 102 L 220 110 H 250" stroke="#b5a594" stroke-width="1.4"/>
<!-- worksheet -->
<path d="M 386 68 H 434 V 124 H 386 Z" stroke="#4e4338" stroke-width="1.5"/>
<path d="M 394 84 L 399 90 L 408 78" stroke="#6b7a52" stroke-width="1.8"/>
<path d="M 394 96 L 403 105 M 403 96 L 394 105" stroke="#a8543f" stroke-width="1.8"/>
<path d="M 394 116 L 399 121 L 408 110" stroke="#6b7a52" stroke-width="1.8"/>
<path d="M 414 85 H 427 M 414 101 H 427 M 414 116 H 427" stroke="#b5a594" stroke-width="1.3"/>
<!-- loop -->
<path d="M 570 76 A 22 22 0 1 1 550 89" stroke="#9a7048" stroke-width="1.8" stroke-dasharray="5 4"/>
<path d="M 547 100 L 550 88 L 561 93" stroke="#9a7048" stroke-width="1.8"/>
</g>
<text x="90" y="182" class="hw" font-size="19" text-anchor="middle">read everything</text>
<text x="90" y="204" class="hw-s" font-size="16" text-anchor="middle">pretraining</text>
<text x="250" y="182" class="hw" font-size="19" text-anchor="middle">pick a major</text>
<text x="250" y="204" class="hw-s" font-size="16" text-anchor="middle">mid-training</text>
<text x="410" y="182" class="hw" font-size="19" text-anchor="middle">watch, then practice</text>
<text x="410" y="204" class="hw-s" font-size="16" text-anchor="middle">post-training</text>
<text x="570" y="182" class="hw" font-size="19" text-anchor="middle" style="fill:#9a7048">keep learning?</text>
<text x="570" y="204" class="hw-s" font-size="16" text-anchor="middle">continual learning</text>
</svg>
<figcaption>The pipeline, drawn as a path through school. The last stop is still an open road.</figcaption>
</figure>

I'm not suggesting these stages are equivalent to human learning, or that more memory automatically produces better intelligence.

## Where does memory end and reasoning begin?

But this analogy makes me curious about where memory ends and reasoning begins.

If someone could remember everything, would she naturally become better at connecting ideas and discovering new patterns? Or does that require something fundamentally different from memory itself?

And if we could build AI systems that not only remember past interactions, but continuously learn from them, would their abilities develop differently over time?

Perhaps there's an interesting feedback loop here: memory gives us more experiences to reason over, reasoning helps us extract useful lessons, and those lessons shape how we learn from future experiences.

<figure class="post-figure post-figure--doodle">
<svg viewBox="0 0 660 255" xmlns="http://www.w3.org/2000/svg" role="img" aria-label="A doodle of a loop with three bubbles: memory, reasoning and lessons. Memory gives reasoning more to reason over, reasoning finds what is worth keeping, and lessons shape what we learn next. A note in the middle asks whether the loop levels off.">
<defs>
<filter id="roughC" x="-5%" y="-5%" width="110%" height="110%"><feTurbulence type="fractalNoise" baseFrequency="0.03" numOctaves="2" seed="21" result="n"/><feDisplacementMap in="SourceGraphic" in2="n" scale="2.2" xChannelSelector="R" yChannelSelector="G"/></filter>
</defs>
<style>.hw{font-family:'Caveat','Cormorant Garamond',cursive;fill:#4e4338}.hw-s{font-family:'Caveat','Cormorant Garamond',cursive;fill:#8a7b6b}</style>
<g filter="url(#roughC)" fill="none" stroke-linecap="round" stroke-linejoin="round">
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
<text x="330" y="59" class="hw" font-size="22" text-anchor="middle">memory</text>
<text x="482" y="195" class="hw" font-size="22" text-anchor="middle">reasoning</text>
<text x="178" y="195" class="hw" font-size="22" text-anchor="middle">lessons</text>
<text x="472" y="98" class="hw-s" font-size="17">more to reason over</text>
<text x="330" y="244" class="hw-s" font-size="17" text-anchor="middle">what's worth keeping</text>
<text x="190" y="98" class="hw-s" font-size="17" text-anchor="end">shapes what we learn next</text>
<text x="330" y="150" class="hw" font-size="18" text-anchor="middle" style="fill:#a8543f">or does it level off?</text>
</svg>
<figcaption>Memory feeds reasoning, and reasoning decides what is worth remembering.</figcaption>
</figure>

Or perhaps there are limits to how much intelligence can emerge from accumulated experience alone.

## A learner we've never met before

But there's another part of this analogy that I find fascinating.

Throughout human history, we've never really encountered a learner with anything close to this kind of capacity.

Perhaps training AI is, in some ways, like teaching an extraordinarily gifted child: one who has absorbed more knowledge than any human could in a lifetime, yet whose learning process is fundamentally different from ours.

And if that's the case, how we choose to teach such a learner might matter enormously, not only for what it can do, but also for how it approaches problems, makes decisions, and responds to the world.

I remember reading about Amanda Askell at Anthropic, whose work involves shaping Claude's character. It is essentially thinking about how to teach an AI to be good, honest, and helpful.

I find that perspective particularly interesting. We're not just teaching models to solve math problems or write better code. We're also making choices about which behaviors to encourage, what kinds of feedback to provide, and what values we want them to reflect.

And just as education can shape a gifted child in both good and bad ways, perhaps our training choices can have consequences we don't fully anticipate.

It brings me back to the question I started with.

If we have never encountered a human with this kind of memory and learning capacity, how much do we really understand about the best way to teach one?

And how much of what AI eventually becomes will be shaped not just by how much it can learn, but by how we choose to teach it?

I don't have answers yet. But these feel like the kind of questions that could keep AI researchers excited about the work for some time.
