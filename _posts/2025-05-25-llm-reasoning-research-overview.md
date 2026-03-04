---
title: "Reasoning in Large Language Models: A Research-Centric Overview"
date: 2025-05-25
categories:
  - AI Learning Guide
  - AI Research
  - Large Language Models
  - Machine Learning
tags:
  - reasoning
  - large language models
  - chain of thought
  - research overview
  - transformers
read_time: "15-25 min read"
---

"Can LLMs reason?" is one of those questions that generates more heat than light, mostly because people mean very different things by "reasoning." This post tries to cut through that by grounding the discussion in concrete technical definitions, drawing heavily from a CS25 lecture by Denny Zhou (Google DeepMind) ([YouTube](https://www.youtube.com/watch?v=ebnX5Ur1hBk)) and the surrounding research literature.

The short version: pretrained models already have latent reasoning abilities—the research challenge is figuring out how to surface and sharpen them.

---

## 1. What Do We Mean by "Reasoning"?

Zhou makes a practical move that sidesteps the philosophical debate entirely. His working definition:

*Reasoning in LLMs = generating intermediate tokens (steps) between input and final output.*

That's it. No claims about "understanding" or "consciousness"—just: does the model produce step-by-step work that improves its final answer? This is concrete, measurable, and turns out to align well with both theory and experiment.

A quick example with the last-letter concatenation task:
- Input: *"artificial intelligence"*
- Without reasoning: model outputs `LE` (or gets it wrong)
- With reasoning: *"The last letter of artificial is L. The last letter of intelligence is E. Concatenating gives LE."*

Same answer, but the reasoning trace makes it robust to harder instances where direct prediction fails.

---

## 2. Why It Works: Theoretical Grounding

There's a clean theoretical story here, not just empirical tricks:

**Boolean circuit complexity (Zhou et al. + Stanford):** Any function computable by Boolean circuits of size *T* can be solved by a constant-size transformer generating *O(T)* intermediate tokens. In other words, intermediate steps serve as computational depth surrogates—they give a fixed-depth transformer the ability to simulate deeper computation.

This is a satisfying result because it tells you *why* chain-of-thought helps: the model is literally doing more computation per problem, using the generated tokens as scratchpad.

**Neuro-symbolic precedents:** The idea of breaking complex problems into intermediate steps isn't new—it mirrors program synthesis and symbolic AI, where reasoning chains reduce combinatorial explosion.

**Early work:** DeepMind introduced natural language intermediate tokens for math problem solving back in 2017 ([arXiv:1706.01340](https://arxiv.org/abs/1706.01340)), before the current wave of interest.

---

## 3. Techniques for Inducing Reasoning

### 3.1 Chain-of-Thought (CoT) Prompting

The technique that kicked off the current wave. Wei et al. ([2022](https://arxiv.org/abs/2201.11903)) showed that adding step-by-step exemplars to the prompt—or even just appending *"Let's think step by step"*—reshapes the output distribution toward reasoning traces. Simple, effective, and requires no model changes.

The limitation: it's sensitive to prompt design, and the quality of reasoning varies. You're relying on the model's pretrained reasoning ability and just nudging it to show its work.

### 3.2 Self-Consistency

Wang et al. ([2022](https://arxiv.org/abs/2203.11171)) had an elegant idea: sample multiple reasoning paths, then take a majority vote on the final answer. Different reasoning paths might make different mistakes, but the correct answer tends to show up more often.

```
Algorithm SelfConsistency(Prompt x, Model M, Samples N):
    answers = []
    for i in 1..N do
        reasoning, answer = Sample(M, x)
        answers.append(answer)
    return Mode(answers)
```

This works surprisingly well—significant accuracy gains on math and commonsense benchmarks. The trade-off is compute: you're running the model N times per question. In practice, N=5 to N=40 depending on the task and budget.

### 3.3 Decoding Strategies

An underappreciated finding: greedy decoding actively suppresses reasoning. CoT decoding expands the search beyond the top-1 token at each step and selects outputs based on final-answer confidence. The reasoning ability is already latent in the pretrained model—greedy decoding just never lets it surface.

### 3.4 Supervised Fine-Tuning (SFT)

Train directly on human-annotated reasoning traces (e.g., GSM8K, [Cobbe et al., 2021](https://arxiv.org/abs/2110.14168)). Gets strong performance on the training distribution but generalizes poorly—models tend to memorize reasoning *patterns* rather than learning to reason *flexibly*. Also, human annotation of reasoning traces is expensive and hard to scale.

### 3.5 Reinforcement Learning Fine-Tuning (RLFT / RLAIF)

The STAR approach (Self-Taught Reasoner, [Zelikman et al., 2022](https://arxiv.org/abs/2203.14465)) is a cleaner alternative: the model generates its own reasoning traces, a verifier checks whether the final answer is correct, and the model is updated on the successful traces. Iterate.

```
Algorithm RLFT(Prompt x, Model M, Verifier V, Iterations T):
    for t in 1..T do
        reasoning, answer = Sample(M, x)
        if V(answer) == correct then
            Update(M, reasoning, answer)
    return M
```

This generalizes better than SFT because the model explores its own reasoning space rather than imitating human traces. The catch: you need a reliable verifier, which limits this to domains with checkable answers (math, code, logic). Extending it to open-ended domains is an active research problem.

### 3.6 Scaling Chain-of-Thought Length

A somewhat surprising finding: longer reasoning traces often improve performance more than scaling up parameter count. There are cases where letting a smaller model think for longer beats a larger model that answers immediately. This echoes Sutton's *Bitter Lesson*—compute spent on search/reasoning can substitute for static knowledge.

---

## 4. Retrieval and Aggregation

Two complementary enhancements:

**Aggregation:** Beyond self-consistency, methods like consensus decoding and ensembles (OpenAI's o1 model uses test-time compute scaling along these lines) combine multiple reasoning attempts. The key insight is that diversity of reasoning paths matters more than quality of any single path.

**Retrieval-augmented reasoning:** Instead of reasoning from scratch, retrieve related solved problems or relevant knowledge first, then reason by analogy. Step-back prompts for geometry tasks are a nice example—the model retrieves high-level principles before diving into the specific problem. Retrieval quality becomes the bottleneck here, but when it works, it's a meaningful boost.

---

## 5. Method Comparison

| Method | Core Idea | Strengths | Weaknesses |
|--------|-----------|-----------|------------|
| CoT Prompting | "Let's think step by step" | Zero-cost, no training needed | Fragile; depends on prompt quality |
| Self-Consistency | Sample N paths, majority vote | Big accuracy gains; implicit uncertainty | N× compute cost |
| CoT Decoding | Non-greedy decoding with confidence selection | Surfaces latent pretrained reasoning | Heuristic; may miss optimal paths |
| SFT | Train on human reasoning traces | Strong in-distribution | Poor generalization; annotation cost |
| RLFT | Self-generated traces + verifier feedback | Better generalization, scalable | Needs reliable verifier; limited to checkable domains |
| Retrieval + Reasoning | Recall related problems, then reason | Extends to broader knowledge | Retrieval quality is critical |

The practical takeaway: CoT prompting is the default starting point (free and often good enough), self-consistency is worth the compute if accuracy matters, and RLFT is the most promising direction for pushing the frontier—but only where you have verifiers.

---

## 6. Limitations and Open Problems

The honest picture of where things stand:

**Non-verifiable tasks** are the big gap. Math and code have automatic verifiers; creative writing, open-ended analysis, and real-world programming don't. Most of the impressive reasoning results are in domains where you can check the answer, and it's unclear how well these techniques transfer to domains where you can't.

**Hallucinations** remain a problem even with reasoning. A model can produce a confident, well-structured reasoning trace that arrives at a wrong answer—and the trace itself can contain plausible-sounding but incorrect steps. Confidence estimation is imperfect and tends to be poorly calibrated exactly when it matters most (on hard or unusual inputs).

**Benchmark saturation** is a growing concern. Models are approaching ceiling performance on standard reasoning benchmarks (GSM8K, MATH, etc.), which makes it hard to measure progress. The field needs evaluations that test compositional generalization and real-world robustness rather than pattern-matching on familiar problem types.

**Reasoning vs. retrieval:** It's still debated how much of what we call "reasoning" in LLMs is genuine multi-step inference versus sophisticated pattern matching against training data. The Boolean circuit complexity results suggest real computation is happening, but the empirical picture is muddier—models sometimes fail on trivially modified versions of problems they solve easily, which is hard to explain if they're truly reasoning.

---

## 7. Where This Is Going

**Beyond verifiable tasks:** Extending RLFT-style training to subjective and open-ended domains. This probably requires better proxy verifiers—maybe LLM-as-judge approaches, or human feedback at coarser granularity.

**Scaling along new dimensions:** Instead of just scaling parameters, scale reasoning length, verifier accuracy, and retrieval breadth. OpenAI's o1 and subsequent reasoning models are early examples of scaling test-time compute rather than model size.

**Hybrid neuro-symbolic systems:** Combining LLMs with external symbolic tools for tasks that need precise multi-step logic. Logic-LM ([Xu et al., 2023](https://arxiv.org/abs/2305.12295)) is one example, using LLMs to translate natural language into formal logic and then running a solver.

**Application-level reasoning:** Moving from benchmarks to real deployments—coding assistants, theorem proving, medical diagnosis, decision support. The gap between benchmark reasoning and useful-in-practice reasoning is nontrivial.

---

## Key References

- Wei et al., 2022 – Chain-of-Thought Prompting ([arXiv:2201.11903](https://arxiv.org/abs/2201.11903))
- Wang et al., 2022 – Self-Consistency ([arXiv:2203.11171](https://arxiv.org/abs/2203.11171))
- Cobbe et al., 2021 – GSM8K ([arXiv:2110.14168](https://arxiv.org/abs/2110.14168))
- Zelikman et al., 2022 – STAR ([arXiv:2203.14465](https://arxiv.org/abs/2203.14465))
- Xu et al., 2023 – Logic-LM ([arXiv:2305.12295](https://arxiv.org/abs/2305.12295))
- Reasoning in LLMs: A Geometric Perspective, 2024 ([arXiv:2407.02678](https://arxiv.org/abs/2407.02678))
- Survey of Reasoning in LLMs, 2025 ([arXiv:2502.17419](https://arxiv.org/abs/2502.17419))
- Sutton, 2019 – *The Bitter Lesson* ([link](http://www.incompleteideas.net/IncIdeas/BitterLesson.html))

---

## Summary

LLM reasoning is best understood as probabilistic generation of intermediate steps—not symbolic deduction, but something that produces similar results in many settings. The theoretical grounding (constant-size transformers simulating arbitrary circuits via intermediate tokens) gives a clean explanation for *why* step-by-step prompting works.

In practice, the methods form a clear progression: CoT prompting is the easy starting point, self-consistency trades compute for accuracy, and RLFT pushes the frontier by having models learn from their own successful reasoning attempts. The main bottleneck is the verifiability constraint—most strong results are in domains where you can automatically check the answer, and extending to open-ended tasks remains the central open problem.

> *"The truth always turns out to be simpler than you thought."* — Richard Feynman

---

{% include citation.html
  title="Reasoning in Large Language Models: A Research-Centric Overview"
  author="Renee Jia"
  journal="renee-jia.github.io"
  year="2025"
  url="https://renee-jia.github.io/ai-learning-guide/llm-reasoning-research-overview/"
  bibtex_key="reneejia2025llmreasoning"
%}
