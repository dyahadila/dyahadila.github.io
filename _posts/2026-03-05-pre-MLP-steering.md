---
layout: post
title: "The Case for Pre-MLP Steering"
date: 2026-03-05 09:00:00 -0600
description: "interesting stufs we learned about pre-MLP steering"
tags: [research]
categories: [blog]
---

Recently, we published [our paper](https://arxiv.org/abs/2603.00425) [1] on principled activation steering (if you haven't, I recommend checking out the paper! it is the PhD work I am most proud of :wink:). We found that post-block steering — after the residual connection is added back to the MLP output — is the theoretically backed, most expressive steering site. If your goal is replicating weight-based fine-tuning with orders of magnitude less parameters, post-block is the way to go.

But that doesn't mean pre-MLP steering isn't interesting. During our analysis, we discovered that pre-MLP steering (while less expressive for fine-tuning-like effects) reveals something interesting about how models process context. In fact, it has a neat connection to in-context learning. In this blog we are going to discuss interesting things we learned about pre-MLP steering during the project. Mainly:

1. Pre-MLP steering has a strong connection with in-context learning (ICL).
2. The choice of a model's *activation function* plays a role into how steerable a model is when we do pre-MLP steering


## Pre-MLP steering formulation

First let's set up some notation and define what we mean by pre-MLP steering. Following notation in our paper, steering updates a model's activations as follows:

$$h \to h + \delta h$$. 

Pre-MLP steering does this update on activations before the MLP. In our experiments we steer the output of attention (red text arrow below), but there are works that steer specific attention heads like those in [2, 3, 4]. Note: we have only checked the derivation and experiments in this blog on pre-MLP steering like our formulation, we have not analyzed if the same math checks out for per-head steering.

<div style="display: flex; justify-content: center; margin: 2rem 0;">
  <img src="{{ '/assets/img/pre_mlp_blog/steering_sites.png' | relative_url }}" alt="Jintervention sites" style="width:60%; height: auto;" />
</div>

## In-Context Learning and Pre-MLP Steering are closely related

TLDR: *What ICL does implicitly — shifting attention outputs based on context — is what pre-MLP steering does explicitly*

Consider a single Transformer block $$T$$ with an input $$h$$ (the residual stream, usually the output from the previous layer):

$$T(h) = h + \text{Attn}(h) + \text{MLP}(h + \text{Attn}(h))$$

(We omit LayerNorm for clarity, but the argument holds with it included.)

The attention output gets added to the residual stream, and the MLP receives the sum $$h + \text{Attn}(h)$$ as its input. Now, what happens when we add context $$C$$ (e.g., a few input-output examples) to the prompt? The attention layer is the only component that directly accesses other tokens in the sequence, as the MLP processes each position independently. So context enters the block through attention, and everything downstream reacts to that shifted input:

$$T(C, h) = h + \text{Attn}(C, h) + \text{MLP}(h + \text{Attn}(C, h))$$

The attention output shifts: $$\text{Attn}(h) \mapsto \text{Attn}(C, h)$$, and this shifted value feeds into both the skip connection and the MLP. 

From the MLP's perspective, its input changed from $$h + \text{Attn}(h)$$ to $$h + \text{Attn}(C, h)$$. We can rewrite this as:

$$h + \text{Attn}(C, h) = \underbrace{h + \text{Attn}(h)}_{\text{original MLP input}} + \underbrace{\text{Attn}(C, h) - \text{Attn}(h)}_{\Delta_A}$$

This is exactly pre-MLP steering with $$\delta h = \Delta_A$$. The attention mechanism computes the steering vector for us: ICL *is* pre-MLP steering (when we set $$\delta h$$ to be $$\Delta_A$$), where the context determines the direction. This observation is straightforward, but there's a deeper result: Dherin et al. 2025 [5] showed this attention shift is equivalent to a rank-1 weight update to the MLP (highly recommend checking this paper out if you haven't, it's awesome).

## On Activation Function's role in pre-MLP steering

Early on, when we started experimenting with different intervention sites (pre-MLP, post-MLP, post-block), we stumbled upon something interesting: *a model's pre-MLP steerability is governed by its choice of activation function*. Our experiments suggest that models with SiLU are steerable across many layers, while models with GELU are only steerable at the very first layer.

In the following figures, we perform pre-MLP steering on individual layers (x-axis) with varying steering strength $$\alpha$$, and measure how much steering moves the model toward a fine-tuned target. Specifically, the y-axis shows:

$$\Delta\text{KL} = \text{KL}(\text{steered} \| \text{finetuned}) - \text{KL}(\text{base} \| \text{finetuned})$$

Negative values mean steering brought the model closer to the fine-tuned model (desirable), zero means no change, and positive means steering made things worse. We plot this for the first 5 generated tokens. The dataset we use is Winogrande [6] formatted as an (A)/(B) multiple choice. So each generation step looks like:

```
Token 1:
Answer: The correct answer is (

Token 2:
Answer: The correct answer is (B

Token 3:
Answer: The correct answer is (B)

Token 4:
Answer: The correct answer is (B) Jennifer

Token 5:
Answer: The correct answer is (B) Jennifer.
```

### Llama (SiLU activation):

<div style="display: flex; justify-content: center; margin: 2rem 0;">
  <img src="{{ '/assets/img/pre_mlp_blog/llama_premlp.png' | relative_url }}" alt="Llama pre-MLP" style="width:100%; height: auto;" />
</div>

### Gemma (GELU activation):

<div style="display: flex; justify-content: center; margin: 2rem 0;">
  <img src="{{ '/assets/img/pre_mlp_blog/gemma_premlp.png' | relative_url }}" alt="Gemma pre-MLP" style="width:100%; height: auto;" />
</div>

**Notice the stark difference in steerability?** Especially on key tokens (like Token 2: A/B) Llama is steerable across many layers (and even more so on middle layers, aligned with the finding from one of the first steering papers [7]); while after layer 0, Gemma is practically unsteerable. 

To confirm this is a pre-MLP phenomenon and not a general property of Gemma, here is Gemma with post-MLP steering:

### Gemma (post-MLP steering)

<div style="display: flex; justify-content: center; margin: 2rem 0;">
  <img src="{{ '/assets/img/pre_mlp_blog/gemma_postmlp.png' | relative_url }}" alt="Gemma post-MLP" style="width:100%; height: auto;" />
</div>

Gemma becomes steerable again — particularly at token 2 and token 4 in the middle layers. 

Note: these are different models, so the activation function isn't the only variable, but the post-MLP control experiment strongly suggests it's the key factor.

### Why? A closer look into activation function

Pre-MLP steering has to propagate through the MLP's nonlinearity, while post-MLP steering bypasses it entirely. From [our paper](https://arxiv.org/abs/2603.00425), the output shift caused by pre-MLP steering:

$$
\Delta \text{GLU}_{\text{steer}}(h) = 
W_d\Big[
\underbrace{(\phi'(a_g)\odot a_u)\odot (W_g \Delta h)}_{\text{gated path}}
+
\underbrace{\phi(a_g)\odot (W_u \Delta h)}_{\text{un-gated path}}
\Big] 
+ O(\|\Delta h\|^2)
$$


## Final Remarks

Note that the stuffs we discuss in this blog are conclusions drawn from a relatively limited set of experiments and analysis, not as rigorous as ones we did in the paper.

## References

[1] Adila, Dyah, et al. "Weight Updates as Activation Shifts: A Principled Framework for Steering." arXiv preprint arXiv:2603.00425 (2026).

[2] Yin, Fangcong, Xi Ye, and Greg Durrett. "Lofit: Localized fine-tuning on llm representations." Advances in neural information processing systems 37 (2024): 9474-9506.

[3] Li, Kenneth, et al. "Inference-time intervention: Eliciting truthful answers from a language model." Advances in Neural Information Processing Systems 36 (2023): 41451-41530.

[4] Todd, Eric, et al. "Function vectors in large language models." arXiv preprint arXiv:2310.15213 (2023).

[5] Dherin, Benoit, et al. "Learning without training: The implicit dynamics of in-context learning." arXiv preprint arXiv:2507.16003 (2025).

[6] Sakaguchi, Keisuke, et al. "Winogrande: An adversarial winograd schema challenge at scale." Communications of the ACM 64.9 (2021): 99-106.

[7] Rimsky, Nina, et al. "Steering llama 2 via contrastive activation addition." Proceedings of the 62nd Annual Meeting of the Association for Computational Linguistics (Volume 1: Long Papers). 2024.