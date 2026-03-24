---
title: "Deep Learning Era of Ranking & Recommendation Systems: Must-Read Papers (2016–2020)"
excerpt: "Explore how deep learning transformed ranking and recommendation systems, from Wide & Deep to BERT4Rec, covering core technologies powering platforms like YouTube, Facebook, and Amazon"
categories:
  - AI Learning Guide
  - Machine Learning
  - Recommendation Systems
tags:
  - deep learning
  - neural networks
  - ranking
  - recommendation systems
  - transformers
  - sequential modeling
last_modified_at: 2025-01-20T08:00:00-05:00
read_time: "15-25 min read"
layout: distill
---

The deep learning revolution fundamentally transformed how we build ranking and recommendation systems. Between 2016 and 2020, neural networks moved from research labs to production systems, enabling platforms like YouTube, Facebook, and Amazon to deliver personalized experiences at unprecedented scale. This post explores the key papers that defined this era, from Google's Wide & Deep architecture to the transformer-based sequential models that still power today's recommendation engines.  

---

## 1. Wide & Deep Models  

- [**Wide & Deep Learning for Recommender Systems** – Cheng et al., 2016 (Google)](https://arxiv.org/abs/1606.07792)  
  *Contribution*: Combined **memorization (wide)** and **generalization (deep)** in one model.  
  *Impact*: Deployed in Google Play app recommendation. Inspired many hybrid RecSys models.  

- [**DeepFM: A Factorization-Machine based Neural Network** – Guo et al., 2017](https://arxiv.org/abs/1703.04247)  
  *Contribution*: Unified **FM** and **DNN** for feature interactions.  
  *Impact*: Became one of the most widely adopted CTR prediction models in ads.  

- [**xDeepFM: Combining Explicit and Implicit Feature Interactions** – Lian et al., 2018](https://arxiv.org/abs/1803.05170)  
  *Contribution*: Modeled **explicit (cross-product)** and **implicit (neural)** interactions simultaneously.  
  *Impact*: Improved performance over DeepFM, influential in ad-tech pipelines.  

---

## 2. Neural Collaborative Filtering (NCF)  

- [**Neural Collaborative Filtering** – He et al., WWW 2017](https://arxiv.org/abs/1708.05031)  
  *Contribution*: Replaced MF's dot product with deep neural networks.  
  *Impact*: Marked the mainstream adoption of deep learning in RecSys.  

---

## 3. Sequential & Context-Aware Models  

- [**GRU4Rec: Session-based Recommendations with Recurrent Neural Networks** – Hidasi et al., ICLR 2016](https://arxiv.org/abs/1511.06939)  
  *Contribution*: Introduced **RNNs (GRU)** for session-based recommendation.  
  *Impact*: First demonstration that sequence modeling boosts short-session recsys.  

- [**SASRec: Self-Attentive Sequential Recommendation** – Kang & McAuley, 2018](https://arxiv.org/abs/1808.09781)  
  *Contribution*: Brought **transformers** into recommendation.  
  *Impact*: Hugely influential — became the standard baseline for sequential RecSys.  

- [**BERT4Rec: Sequential Recommendation with Bidirectional Encoder Representations** – Sun et al., 2019](https://arxiv.org/abs/1904.06690)  
  *Contribution*: Adapted **BERT's bidirectional attention** for RecSys.  
  *Impact*: Achieved state-of-the-art sequential recommendation performance.  

---

## 4. Industry-Scale Systems  

- [**Practical Lessons from Predicting Clicks on Ads at Facebook** – He et al., ADKDD 2014](https://research.facebook.com/publications/practical-lessons-from-predicting-clicks-on-ads-at-facebook/)  
  *Contribution*: Shared lessons from production-scale CTR prediction.  
  *Impact*: Still one of the most-cited industrial RecSys papers.  

- [**YouTube Recommendations: Deep Neural Networks for Large-Scale Recommendation Systems** – Covington et al., RecSys 2016](https://storage.googleapis.com/pub-tools-public-publication-data/pdf/45530.pdf)  
  *Contribution*: Introduced the now-standard **two-tower architecture** (candidate generation + ranking).  
  *Impact*: Hugely influential, shaping large-scale RecSys pipelines.  

---

## Why These Papers Matter  

1. **Deep learning enters RecSys**: Moving from latent factors to neural networks.  
2. **Sequence awareness**: First use of RNNs and Transformers to model **temporal user behavior**.  
3. **Industry adoption**: These architectures (Wide & Deep, YouTube's two-tower, Facebook CTR) are still in production today.  

---

## Suggested Reading Order  

1. **Wide & Deep → DeepFM → xDeepFM** (feature interaction evolution).  
2. **NCF** (neural replacement of MF).  
3. **GRU4Rec → SASRec → BERT4Rec** (sequential models).  
4. **YouTube 2016 + Facebook 2014** (industry blueprints).  



---

{% include citation.html 
  title="Deep Learning Era of Ranking & Recommendation Systems: Must-Read Papers (2016–2020)"
  author="Renee Jia"
  journal="renee-jia.github.io"
  year="2025"
  url="https://renee-jia.github.io/ai-learning-guide/deep-learning-era-ranking-recommendation-systems/"
  bibtex_key="reneejia2025deeplearning"
%}

