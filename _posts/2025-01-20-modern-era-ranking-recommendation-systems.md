---
title: "Contemporary RecSys: Industry-Scale Architectures & Multimodal Systems (2020–2025)"
date: 2025-01-20
categories:
  - AI Learning Guide
  - AI Research
  - Recommendation Systems
  - Machine Learning
tags:
  - ranking
  - recommendation
  - deep learning
  - transformers
  - multitask learning
  - multimodal
  - graph neural networks
read_time: "15-25 min read"
---

# Contemporary RecSys: Industry-Scale Architectures & Multimodal Systems (2020–2025)

This era represents a pivotal shift toward production-ready, billion-scale architectures that power today's major platforms. This comprehensive guide covers the essential papers that define contemporary RecSys, from industry-standard models like DLRM and DCN v2 to cutting-edge advances in multitask learning, transformers, and multimodal recommendation.

This era is defined by:  
- **Industry-scale recommendation architectures** (DLRM, DCN v2).  
- **Multitask learning** for ads, feeds, and conversion modeling.  
- **Transformers for RecSys sequences**.  
- **Multimodal recommendation** (text, image, video).  
- **Graph neural networks for recommendations**.  
- **Efficiency & serving optimizations** for billion-scale systems.  

---

## 1. Large-Scale Deep RecSys Architectures  

- [**DLRM: Deep Learning Recommendation Model** – Naumov et al., 2019 (Facebook)](https://arxiv.org/abs/1906.00091)  
  *Contribution*: Introduced **embedding tables + MLP interaction layers** optimized for RecSys.  
  *Impact*: Standard production RecSys model for CTR prediction. Still widely deployed.  

- [**Deep & Cross Network (DCN)** – Wang et al., 2017 → [DCN v2 (2021)](https://arxiv.org/abs/2008.13535)  
  *Contribution*: Explicitly modeled **feature crosses** in a learnable way.  
  *Impact*: Highly effective in ad-ranking pipelines (Google).  

- [**Deep Interest Network (DIN)** – Zhou et al., 2018 (Alibaba)](https://arxiv.org/abs/1706.06978)  
  *Contribution*: Used attention to capture **user interests** from behavior sequences.  
  *Impact*: Deployed at Alibaba Taobao, strong influence on RecSys attention models.  

- [**Deep Interest Evolution Network (DIEN)** – Zhou et al., 2019](https://arxiv.org/abs/1809.03672)  
  *Contribution*: Modeled **temporal evolution of user interests**.  
  *Impact*: Pioneered modeling evolving preferences, key in e-commerce.  

---

## 2. Multitask Learning in Recommendation  

- [**MMoE: Multi-gate Mixture-of-Experts** – Ma et al., KDD 2018](https://dl.acm.org/doi/10.1145/3219819.3220007)  
  *Contribution*: Shared experts with task-specific gates for **CTR, CVR, dwell-time prediction**.  
  *Impact*: Widely used in ads & feeds.  

- [**ESMM: Entire Space Multi-task Model** – Ma et al., 2018](https://arxiv.org/abs/1804.07931)  
  *Contribution*: Solved **sparse CVR data** by coupling CTR + CVR prediction.  
  *Impact*: Crucial in conversion modeling.  

- [**PLE: Progressive Layered Extraction for Multitask Learning** – Tang et al., KDD 2020](https://dl.acm.org/doi/10.1145/3394486.3403279)  
  *Contribution*: Improves on MMoE by separating **task-specific vs shared knowledge**.  
  *Impact*: State-of-the-art for multi-objective ranking.  

---

## 3. Transformers & Sequential Models  

- [**SASRec: Self-Attentive Sequential Recommendation** – Kang & McAuley, 2018](https://arxiv.org/abs/1808.09781)  
  *Contribution*: First transformer-based RecSys.  
  *Impact*: Widely adopted as a baseline.  

- [**BERT4Rec: Sequential Recommendation with Bidirectional Encoder Representations** – Sun et al., 2019](https://arxiv.org/abs/1904.06690)  
  *Contribution*: Adapted **BERT** for RecSys sequences.  
  *Impact*: Set the stage for pretraining + fine-tuning RecSys.  

- [**UniSRec / UPRec (Universal Sequence Representations for RecSys)** – 2020+](https://arxiv.org/abs/2205.08084)  
  *Contribution*: Pretrained sequence models on cross-domain data.  
  *Impact*: Early step towards **RecSys foundation models**.  

---

## 4. Multimodal Recommendation  

- [**CLIP: Learning Transferable Visual Models from Natural Language Supervision** – Radford et al., 2021](https://arxiv.org/abs/2103.00020)  
  *Contribution*: Joint **image-text embeddings** via contrastive training.  
  *Impact*: Foundation for multimodal retrieval.  

- [**CoCa: Contrastive Captioners are Image-Text Foundation Models** – Yu et al., 2022](https://arxiv.org/abs/2205.01917)  
  *Contribution*: Unified contrastive + generative pretraining for multimodal tasks.  
  *Impact*: Shows multimodal RecSys moving beyond retrieval to generation.  

- [**VideoCLIP & MMRecSys** (2022–2023)]  
  *Contribution*: Extended contrastive learning to **video + audio + text**.  
  *Impact*: Practical for TikTok, YouTube, and video-centric feeds.  

---

## 5. Graph-based Recommendation  

- [**PinSage: Graph Convolutional Neural Networks for Web-Scale Recommender Systems** – Ying et al., KDD 2018](https://arxiv.org/abs/1806.01973)  
  *Contribution*: Scalable GNN-based recommendation at Pinterest.  
  *Impact*: First GNN RecSys at billion-scale.  

- [**GraphRec, LightGCN, NGCF (2019–2021)**]  
  *Contribution*: Applied graph neural networks to user–item bipartite graphs.  
  *Impact*: Became strong baselines for collaborative filtering.  

- [**PinnerFormer: Transformer Architectures for Sequential Recommendation at Pinterest** – 2022](https://arxiv.org/abs/2205.09272)  
  *Contribution*: Combined graph + transformer signals.  
  *Impact*: Practical large-scale deployment.  

---

## 6. Efficiency & Serving at Scale  

- [**TFX: TensorFlow Extended** – 2017+](https://www.tensorflow.org/tfx)  
  *Impact*: Standard pipeline for RecSys feature engineering + training + serving.  

- [**HugeCTR (NVIDIA)** – 2020+](https://github.com/NVIDIA/HugeCTR)  
  *Contribution*: GPU-optimized framework for RecSys training.  
  *Impact*: Critical for billion-parameter embedding tables.  

- [**DeepRec (Alibaba)** – 2021+](https://github.com/alibaba/DeepRec)  
  *Contribution*: Open-source RecSys framework optimized for **large embeddings + distributed training**.  
  *Impact*: Shows industry emphasis on RecSys infrastructure.  

---

## 7. Key Shifts in This Era  

1. **Industry architectures**: DLRM & DCN v2 became RecSys "backbones."  
2. **Multi-task models**: MMoE, ESMM, PLE → optimizing CTR, CVR, dwell, engagement jointly.  
3. **Transformers everywhere**: SASRec, BERT4Rec, UniRec → sequential & pretraining.  
4. **Multimodal RecSys**: CLIP & CoCa → text, image, video embeddings.  
5. **Graph-based models**: PinSage & LightGCN → structure-aware recommendation.  
6. **Efficiency & serving**: Specialized frameworks (HugeCTR, DeepRec) to handle **billion-scale embeddings**.  

---

## Suggested Reading Order  

1. **DLRM → DCN v2** → modern RecSys architecture design.  
2. **MMoE → ESMM → PLE** → multi-task advances.  
3. **SASRec → BERT4Rec → UniRec** → transformers in RecSys.  
4. **CLIP → CoCa → VideoCLIP** → multimodal retrieval & recommendation.  
5. **PinSage → LightGCN → PinnerFormer** → graph-enhanced RecSys.  
6. **HugeCTR / DeepRec** → infrastructure for large-scale serving.  

---

{% include citation.html 
  title="Modern Era of Ranking & Recommendation Systems: Must-Read Papers (2020–2025)"
  author="Renee Jia"
  journal="renee-jia.github.io"
  year="2025"
  url="https://renee-jia.github.io/ai-learning-guide/modern-era-ranking-recommendation-systems/"
  bibtex_key="reneejia2025modern"
%}
