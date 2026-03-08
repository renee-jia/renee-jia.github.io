---
title: "Modeling Long User Histories for Ads Ranking"
excerpt: "How ads ranking systems went from aggregated feature counts to retrieve-and-compress architectures that handle 10,000+ user events under millisecond latency constraints."
date: 2026-03-07
categories:
  - AI Learning Guide
  - AI Research
  - Recommendation Systems
  - Machine Learning
tags:
  - ads ranking
  - CTR prediction
  - long sequence modeling
  - attention mechanisms
  - user behavior modeling
  - deep learning
  - recommendation systems
last_modified_at: 2026-03-07T08:00:00-08:00
read_time: "20-30 min read"
toc: false
---

## Introduction

Ads ranking systems predict user responses—click-through rate (CTR), conversion rate (CVR), engagement—and the strongest signal is usually the user's own behavioral history: what they clicked, browsed, purchased, and ignored.

The catch is that these histories get long. Active users generate thousands of events. Encoding those sequences at inference time, within the latency budget of a real-time ad auction (single-digit milliseconds), requires co-design of model architecture and serving infrastructure. That tension—expressiveness versus efficiency—has shaped most of the architectural evolution in industrial ads ranking over the past decade.

This post walks through the major generations of that evolution: feature aggregation, attention-based sequence models, and the retrieval-and-compress architectures running in production today.

---

## 1. Feature Aggregation and Wide & Deep Models (2016–2018)

The first generation of deep CTR models didn't bother with sequences. User history got collapsed into aggregate statistics:

- Click and conversion counts over time windows (7-day, 30-day, lifetime)
- Category and advertiser frequency distributions
- Recency features (time since last click in category $c$)
- Cross-features between user segments and ad attributes

These features fed into Wide & Deep architectures, where a linear "wide" component memorized sparse cross-features and a deep MLP learned dense generalizations.

**Representative architecture:**
```
User History → Feature Engineering → [Wide (crosses) + Deep (MLP)] → Prediction
```

**Key paper:**
- Cheng et al., ["Wide & Deep Learning for Recommender Systems"](https://arxiv.org/abs/1606.07792) (Google, 2016)

This paradigm scaled well—feature pipelines are embarrassingly parallel, and the model itself is just an MLP. But it throws away all temporal structure. Whether a user clicked on running shoes yesterday or six months ago gets compressed into the same count. Ordering, transitions between interests, context around each interaction—gone.

---

## 2. Attention-Based Sequence Modeling (2018–2019)

The observation behind the next generation is straightforward: not all past behaviors matter equally for the current prediction. If you're ranking an outdoor jacket ad, the user's click on hiking boots last week matters a lot more than their click on a laptop charger yesterday.

### DIN: Candidate-Aware Attention

Deep Interest Network (DIN) operationalized this idea: instead of encoding the entire user history into a single fixed-length vector, compute an attention-weighted sum where the *candidate ad* serves as the query.

Given user behavior embeddings $\{e_1, e_2, \ldots, e_T\}$ and candidate ad embedding $e_a$:

$$v_u = \sum_{t=1}^{T} \alpha(e_t, e_a) \cdot e_t$$

where $\alpha(e_t, e_a)$ is a learned attention function (implemented as a small MLP over the concatenation and element-wise product of $e_t$ and $e_a$).

This is target attention—the representation of the user *changes depending on what ad you're scoring*. A user who browsed both electronics and sportswear gets a different representation when you're ranking a phone case versus a yoga mat.

**Key paper:**
- Zhou et al., ["Deep Interest Network for Click-Through Rate Prediction"](https://arxiv.org/abs/1706.06978) (Alibaba, 2018)

### DIEN: Modeling Interest Evolution

DIN treats user history as a set—it doesn't model the temporal evolution of interests. Deep Interest Evolution Network (DIEN) addressed this by adding a GRU layer to capture sequential dynamics, then applying an auxiliary loss to ensure the hidden states actually track interest evolution:

$$h_t' = \text{AUGRU}(h_{t-1}', e_t, \alpha_t)$$

where AUGRU (Attention-based GRU) gates the update by the attention score $\alpha_t$ with respect to the target ad. This means the sequential encoder is *also* candidate-aware—it tracks the evolution of interest *relevant to the current candidate*.

**Key paper:**
- Zhou et al., ["Deep Interest Evolution Network for Click-Through Rate Prediction"](https://arxiv.org/abs/1809.03672) (Alibaba, 2019)

The limitation of both DIN and DIEN: they work well with sequences of ~50–200 events. Beyond that, the attention computation over the full sequence becomes a latency bottleneck at serving time.

---

## 3. Transformer-Based User Modeling (2019–2021)

RNNs have well-known problems: sequential computation kills parallelism, and long-range dependencies remain hard to learn despite gating. Transformers fix both.

### BST: Transformers in Production

Behavior Sequence Transformer (BST) was among the first to deploy multi-head self-attention over user behavior sequences in a production ads system. The architecture is straightforward:

```
User behavior sequence → Item embeddings + positional encoding
                       → L × Transformer blocks (self-attention + FFN)
                       → Concatenate with other features
                       → MLP → CTR prediction
```

Each Transformer block computes:

$$\text{Attention}(Q, K, V) = \text{softmax}\left(\frac{QK^\top}{\sqrt{d_k}}\right)V$$

BST showed meaningful CTR improvements in Alibaba's production system, handling sequences of ~50–150 recent behaviors.

**Key papers:**
- Chen et al., ["Behavior Sequence Transformer for E-commerce Recommendation in Alibaba"](https://arxiv.org/abs/1905.06874) (DLP-KDD 2019)
- Kang & McAuley, ["Self-Attentive Sequential Recommendation"](https://arxiv.org/abs/1808.09781) (SASRec, ICDM 2018)
- Sun et al., ["BERT4Rec: Sequential Recommendation with Bidirectional Encoder Representations"](https://arxiv.org/abs/1904.06690) (CIKM 2019)

### The Quadratic Wall

The problem everybody already knows: self-attention is $O(n^2)$ in sequence length, both in compute and memory. At 100 events, that's fine. At 1,000, it's expensive. At 10,000, it's not happening within a millisecond latency budget.

This is what motivated the next wave of work. The question stopped being "how do we model sequences?" and became "how do we model *long* sequences without blowing the serving budget?"

---

## 4. Scaling to Long Sequences (2021–2023)

Production ads systems see thousands of user interactions per user. Alibaba reported modeling sequences of 1,000–10,000+ events. At this scale, neither full self-attention nor even linear-time attention over the raw sequence is practical at serving time. The solutions all share a common structure: **reduce first, then model**.

### 4.1 Truncation

The simplest approach: just keep the most recent $N$ events (typically 50–200) and throw the rest away. This works better than you might expect, because recent behaviors tend to dominate CTR prediction. If you clicked on coffee makers five minutes ago, that's probably the strongest signal for ranking a coffee grinder ad.

The obvious downside: long-term interest signals are gone. A user who researched cameras extensively three months ago but hasn't browsed any recently still has latent purchase intent, and truncation misses it. Most systems compensate with long-term aggregate features on the side—basically bringing back the feature engineering from the Wide & Deep era.

### 4.2 Two-Stage Retrieval: SIM

Search-based Interest Model (SIM) from Alibaba is probably the most influential paper in this space. It introduced the **two-stage retrieve-then-attend** pipeline that most subsequent long-sequence work builds on.

**Stage 1 — General Search Unit (GSU):** Given the candidate ad, retrieve a small subset of relevant behaviors from the full user history. This is a *sub-linear* operation—either a hard search (category matching, inverted index) or a soft search (maximum inner product search over embeddings).

**Stage 2 — Exact Search Unit (ESU):** Apply the standard DIN-style target attention over the retrieved subset (typically 50–200 events), producing the final user representation.

```
Full history (10,000 events)
    → GSU: retrieve top-K relevant behaviors (K ≈ 50–200)
        → ESU: target attention over retrieved subset
            → user representation for this candidate
```

The idea is simple: you don't need to attend over the *entire* history. You just need to find the *relevant* subset first. Retrieval is cheap (sub-linear), and attention only runs over a small enough set to stay within latency.

In practice, Alibaba deployed SIM with user histories of 1,000+ events and observed consistent gains over truncated baselines, particularly for categories with long purchase cycles (electronics, furniture, travel).

**Key paper:**
- Pi et al., ["Search-based User Interest Modeling with Lifelong Sequential Behavior Data for Click-Through Rate Prediction"](https://arxiv.org/abs/2006.05639) (CIKM 2020)

### 4.3 Hash-Based Retrieval: ETA and SDIM

SIM's retrieval stage still needs either an inverted index per user or inner-product computation. Two follow-up papers pushed for even cheaper retrieval using locality-sensitive hashing (LSH).

**ETA (End-to-End Target Attention)** replaces the retrieval stage with SimHash: hash both user behavior embeddings and the candidate ad embedding into binary codes, then use Hamming distance to find the top-K nearest behaviors. This is extremely fast—bitwise XOR and popcount—and allows end-to-end training since the hash functions are learned.

**Key paper:**
- Chen et al., ["End-to-End User Behavior Retrieval in Click-Through Rate Prediction Model"](https://arxiv.org/abs/2108.04468) (2021)

**SDIM (Sampling-based Deep Interest Modeling)** takes a different approach: use multi-round hashing to identify behaviors that collide with the target ad in hash space, treating hash collisions as an approximation of attention. No explicit top-K retrieval needed—behaviors that hash to the same bucket as the candidate get attended to.

**Key paper:**
- Cao et al., ["Sampling Is All You Need on Modeling Long-Term User Behaviors for CTR Prediction"](https://arxiv.org/abs/2205.10249) (CIKM 2022)

Both approaches achieve sub-linear retrieval complexity and fit within production latency budgets while modeling sequences of 10,000+ events.

### 4.4 Hierarchical Modeling

Another strategy: process long histories in stages. User behavior naturally clusters into sessions anyway, so lean into that structure.

```
Events → Session encoder → Session representations
       → Cross-session encoder → User representation
```

The session encoder (often a small Transformer or GRU) captures short-term patterns within a browsing session. The cross-session encoder aggregates across sessions to capture long-term interests. Because session representations are much shorter than event sequences, the cross-session encoder operates over a manageable length.

This also maps naturally to system architecture: session-level representations can be precomputed and cached, with only the current session encoded in real time.

### 4.5 Memory-Based Approaches

Some systems maintain explicit long-term memory outside the model:

- **Cached user embeddings:** Periodically compute and store user representations from historical data; combine with real-time features at serving time.
- **User interest vectors:** Maintain a set of interest cluster centroids per user, updated incrementally as new behaviors arrive.
- **Persistent memory features:** Store historical representations in a key-value store, retrieved by category or topic at inference time.

These approaches aren't glamorous—the "model" for long-term interests is basically a lookup plus aggregation, and the neural network only handles recent behavior and cross-feature interactions. But the split works well in practice because long-term interests change slowly. Updating a cached embedding daily or weekly doesn't lose much signal.

---

## 5. Set-Based Sequence Compression (2023–Present)

There's an argument that sequential order doesn't matter much for long-term interest modeling. Over hundreds or thousands of events, the precise ordering probably carries less signal than the overall *distribution* of behaviors.

### Pooling by Multihead Attention (PMA)

The Set Transformer introduced Pooling by Multihead Attention, which learns a fixed number of "seed" vectors that summarize an input set of arbitrary size:

$$\text{PMA}_k(X) = \text{MultiHead}(S, X, X)$$

where $S \in \mathbb{R}^{k \times d}$ is a learnable matrix of $k$ seed vectors (queries), and $X \in \mathbb{R}^{n \times d}$ is the input set. The output is always $k \times d$ regardless of input size $n$.

```python
class PMA(nn.Module):
    def __init__(self, dim, num_heads, num_seeds):
        super().__init__()
        self.seeds = nn.Parameter(torch.randn(1, num_seeds, dim))
        self.attn = nn.MultiheadAttention(dim, num_heads, batch_first=True)

    def forward(self, x):
        # x: (batch, seq_len, dim) → output: (batch, num_seeds, dim)
        seeds = self.seeds.expand(x.size(0), -1, -1)
        out, _ = self.attn(seeds, x, x)
        return out
```

What makes this attractive for ads: model complexity depends on $k$ (typically 4–32), not on the sequence length. A user with 100 events and a user with 10,000 events produce the same-shaped output.

**Key paper:**
- Lee et al., ["Set Transformer: A Framework for Attention-based Permutation-Invariant Input"](https://arxiv.org/abs/1810.00825) (ICML 2019)

### Perceiver and Latent Bottleneck Architectures

DeepMind's Perceiver generalized this idea: use a small set of latent vectors that cross-attend to a large input, then process the latent vectors with self-attention. Because self-attention operates on the latent set (small) rather than the input (large), the computational cost is decoupled from input size.

$$Z = \text{CrossAttention}(\text{latent}, X) \quad \text{then} \quad Z' = \text{SelfAttention}(Z)$$

This pattern—cross-attend to compress, self-attend to refine—has started showing up in production systems that need to squeeze long user histories into fixed-size representations.

**Key paper:**
- Jaegle et al., ["Perceiver: General Perception with Iterative Attention"](https://arxiv.org/abs/2103.03206) (ICML 2021)

---

## 6. Design Trade-offs

Everything in this space is constrained by production realities:

| Factor | Constraint | Typical budget |
|--------|-----------|---------------|
| Latency | Ad auction deadlines | < 10 ms per candidate |
| Sequence length | Active user histories | 1,000–50,000 events |
| Memory | Embedding table size | Terabytes across shards |
| Model capacity | Serving cost per query | Bounded by QPS × cost |
| Freshness | Real-time behavior ingestion | Minutes to hours of delay |

Each architecture above sits at a different point in this trade-off space. Truncation trades accuracy for simplicity. Two-stage retrieval trades system complexity for both accuracy and efficiency. Set-based compression trades sequential information for fixed-cost representations. There's no single winner—the right choice depends on your latency budget, user activity distribution, and how much of your signal comes from long-term versus recent behavior.

---

## 7. What Holds Up in Practice

A few patterns worth noting:

**Recency dominates.** The last few hours of behavior usually carry more predictive signal than the previous month. Time-decay weighting, recency features, and recent-event enrichment are standard everywhere. This is why truncation works as well as it does, and why it's the baseline everyone ends up comparing against.

**Long-term signal is sparse but high-value.** Long-term interests matter most for infrequent or high-consideration categories—the user who researched laptops for two weeks and then went quiet for a month is exactly who you want to reach with a laptop ad. These signals are rare per-user but valuable in aggregate, and systems that capture them do show consistent lift.

**Offline-online decomposition works.** Compute long-term user representations offline (daily or hourly batch jobs) and combine them with real-time features at serving time. It's not fancy, but it's deployed everywhere because it keeps inference fast while still capturing long-term preferences.

**Feature interactions still matter.** Even with a good sequence model, the prediction layer benefits from explicit cross-features between the sequence representation and the candidate ad. DIN's element-wise product between user and ad embeddings wasn't a minor detail—it's consistently important across systems.

---

## 8. Open Problems

**Ultra-long sequences.** Users with 50,000+ events across platforms and years of history. Retrieval-based approaches handle 10K-scale sequences, but an order of magnitude beyond that will need new indexing and compression strategies.

**Multi-objective ranking.** Ads models need to jointly optimize CTR, CVR, engagement, long-term user value, and advertiser ROI. How sequence representations play with multi-task architectures (MMoE, PLE) is still underexplored—most long-sequence papers only look at CTR.

**Non-stationary interests.** User interests shift suddenly (life events) and drift gradually (seasons, trends). Current models snapshot the interest distribution but don't explicitly track how it evolves over long time horizons.

**Cross-domain transfer.** Users interact across search, social, shopping, and video. Modeling the joint sequence across domains means dealing with heterogeneous event types, different feature spaces, and conflicting relevance signals.

---

## 9. Conclusion

The progression looks like:

```
Feature aggregation → Target attention → Self-attention → Retrieve-then-attend → Compress
```

Each generation hit a specific wall. Feature engineering couldn't capture behavioral structure. Fixed-length encodings couldn't capture relevance to the candidate. RNNs couldn't parallelize. Full attention couldn't scale. And brute-force retrieval couldn't meet latency budgets.

The current state of the art boils down to a simple principle: you don't need to model every event in a user's history. You need to *find* the relevant ones quickly and represent them compactly. Whether that principle keeps working as interaction data grows by another order of magnitude is an open question—but so far, the retrieve-and-compress pattern has held up well.

---

## References

### Feature Aggregation Era
- Cheng, H., et al. ["Wide & Deep Learning for Recommender Systems"](https://arxiv.org/abs/1606.07792) (Google, 2016)

### Attention-Based Sequence Modeling
- Zhou, G., et al. ["Deep Interest Network for Click-Through Rate Prediction"](https://arxiv.org/abs/1706.06978) (Alibaba, KDD 2018)
- Zhou, G., et al. ["Deep Interest Evolution Network for Click-Through Rate Prediction"](https://arxiv.org/abs/1809.03672) (Alibaba, AAAI 2019)

### Transformer-Based User Modeling
- Kang, W. C. & McAuley, J. ["Self-Attentive Sequential Recommendation"](https://arxiv.org/abs/1808.09781) (ICDM 2018)
- Sun, F., et al. ["BERT4Rec: Sequential Recommendation with Bidirectional Encoder Representations from Transformer"](https://arxiv.org/abs/1904.06690) (CIKM 2019)
- Chen, Q., et al. ["Behavior Sequence Transformer for E-commerce Recommendation in Alibaba"](https://arxiv.org/abs/1905.06874) (DLP-KDD 2019)

### Long Sequence Modeling
- Pi, Q., et al. ["Search-based User Interest Modeling with Lifelong Sequential Behavior Data for Click-Through Rate Prediction"](https://arxiv.org/abs/2006.05639) (CIKM 2020)
- Chen, Q., et al. ["End-to-End User Behavior Retrieval in Click-Through Rate Prediction Model"](https://arxiv.org/abs/2108.04468) (2021)
- Cao, H., et al. ["Sampling Is All You Need on Modeling Long-Term User Behaviors for CTR Prediction"](https://arxiv.org/abs/2205.10249) (CIKM 2022)

### Set-Based and Compression Architectures
- Lee, J., et al. ["Set Transformer: A Framework for Attention-based Permutation-Invariant Input"](https://arxiv.org/abs/1810.00825) (ICML 2019)
- Jaegle, A., et al. ["Perceiver: General Perception with Iterative Attention"](https://arxiv.org/abs/2103.03206) (ICML 2021)

---

{% include citation.html
  title="Modeling Long User Histories for Ads Ranking"
  author="Renee Jia"
  journal="renee-jia.github.io"
  year="2026"
  url="https://renee-jia.github.io/ai-learning-guide/long-sequence-learning-ads-systems/"
  bibtex_key="reneejia2026longsequence"
%}
