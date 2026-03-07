---
layout: post
title: "The Case for Pre-MLP Steering"
date: 2026-03-05 09:00:00 -0600
description: "interesting stufs we learned about pre-MLP steering"
tags: [research]
categories: [blog]
---

Recently, we published [our paper](https://arxiv.org/abs/2603.00425) [1] on principled activation steering. If you haven't, I recommend checking it out! It is my favorite PhD work :wink:.

We analyze three common steering site choices, <span style="color: red;">pre-MLP</span>, <span style="color: blue;">post-MLP</span>, and <span style="color: green;">post-block</span>, and found that <span style="color: green;">post-block</span> steering — after the residual connection is added back to the MLP output — is the theoretically backed, most expressive steering site. If your goal is replicating weight-based fine-tuning with orders of magnitude less parameters, <span style="color: green;">post-block</span> is the way to go.

<div style="display: flex; justify-content: center; margin: 2rem 0;">
  <img src="{{ '/assets/img/pre_mlp_blog/steering_sites.png' | relative_url }}" alt="Jintervention sites" style="width:50%; height: auto;" />
</div>

But that doesn't mean <span style="color: red;">pre-MLP</span> steering isn't interesting. In this blog we're going to discuss things we learned about <span style="color: red;">pre-MLP</span> steering during the project. Mainly:


1. <span style="color: red;">Pre-MLP</span> steering has a strong connection with in-context learning (ICL).
2. Different models show very different <span style="color: red;">pre-MLP</span> steerability — and the MLP's nonlinearity appears to play a role, though the full picture is still open.


## <span style="color: red;">Pre-MLP</span> steering formulation

First let's set up some notation and define what we mean by <span style="color: red;">pre-MLP</span> steering. Following notation in our paper, steering updates a model's activations as follows:

$$h \to h + \delta h$$. 

<span style="color: red;">Pre-MLP</span> steering does this update on activations before the MLP. In our experiments below we steer the output of attention, but there are works that steer specific attention heads like those in [2, 3, 4]. 

Note: we have only checked the derivation and experiments in this blog on <span style="color: red;">pre-MLP</span> steering like our formulation, we have not analyzed if the same checks out for per-head steering.

## In-Context Learning and <span style="color: red;">Pre-MLP</span> Steering are closely related

TLDR: <b style="color: #B509AC;">What ICL does implicitly — shifting attention outputs based on context — is what <span style="color: red;">pre-MLP</span> steering does explicitly</b>

Consider a single Transformer block $$T$$ with an input $$h$$ (the residual stream, usually the output from the previous layer):

$$T(h) = h + \text{Attn}(h) + \text{MLP}(h + \text{Attn}(h))$$

(We omit LayerNorm for clarity, but the argument holds with it included.)

The attention output gets added to the residual stream, and the MLP receives the sum $$h + \text{Attn}(h)$$ as its input. Now, what happens when we add context $$C$$ (e.g., a few input-output examples) to the prompt? The attention layer is the only component that directly accesses other tokens in the sequence, as the MLP processes each position independently. So context enters the block through attention, and everything downstream reacts to that shifted input:

$$T(C, h) = h + \text{Attn}(C, h) + \text{MLP}(h + \text{Attn}(C, h))$$

The attention output shifts: $$\text{Attn}(h) \mapsto \text{Attn}(C, h)$$, and this shifted value feeds into both the skip connection and the MLP. 

From the MLP's perspective, its input changed from $$h + \text{Attn}(h)$$ to $$h + \text{Attn}(C, h)$$. We can rewrite this as:

$$h + \text{Attn}(C, h) = \underbrace{h + \text{Attn}(h)}_{\text{original MLP input}} + \underbrace{\text{Attn}(C, h) - \text{Attn}(h)}_{\Delta_A}$$

This is exactly <span style="color: red;">pre-MLP</span> steering with $$\delta h = \Delta_A$$. The attention mechanism computes the steering vector for us: ICL *is* <span style="color: red;">pre-MLP</span> steering (when we set $$\delta h$$ to be $$\Delta_A$$), where the context determines the direction. This observation is straightforward, but there's a deeper result: Dherin et al. 2025 [5] showed this attention shift is equivalent to a rank-1 weight update to the MLP (highly recommend checking this paper out if you haven't, it's awesome).

## Why are some models harder to steer <span style="color: red;">pre-MLP</span>?

Early on, when we started experimenting with different intervention sites, we stumbled upon something interesting: <b style="color: #B509AC;">different models show very different <span style="color: red;">pre-MLP</span> steerability</b>. In our experiments, Llama (which uses SiLU) is steerable across many layers, while Gemma (which uses GELU) is practically unsteerable after layer 0 — and the MLP's nonlinearity appears to be involved.

In the following figures, we perform <span style="color: red;">pre-MLP</span> steering on individual layers (x-axis) using the oracle steering vector $\delta h_{\text{oracle}} = h_{\text{finetuned}} - h_{\text{base}}$ — literally replacing the base model's activation with the fine-tuned model's activation at that layer. We scale this vector by a steering strength $\alpha$ and measure how much the steered model's output moves toward the fine-tuned target. Specifically, the y-axis shows:

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

### Llama <span style="color: red;">pre-MLP</span> steering (SiLU activation):

<div style="display: flex; justify-content: center; margin: 2rem 0;">
  <img src="{{ '/assets/img/pre_mlp_blog/llama_premlp.png' | relative_url }}" alt="Llama pre-MLP" style="width:100%; height: auto;" />
</div>

### Gemma <span style="color: red;">pre-MLP</span> steering (GELU activation):

<div style="display: flex; justify-content: center; margin: 2rem 0;">
  <img src="{{ '/assets/img/pre_mlp_blog/gemma_premlp.png' | relative_url }}" alt="Gemma pre-MLP" style="width:100%; height: auto;" />
</div>

**Notice the stark difference in steerability?** Especially on key tokens (like Token 2: A/B, Token 4: the answer string) Llama is steerable across many layers (and even more so on middle layers, aligned with the finding from one of the first steering papers [7]); while after layer 0, Gemma is practically unsteerable. 

To confirm this is a <span style="color: red;">pre-MLP</span> phenomenon and not a general property of Gemma, here is Gemma with <span style="color: blue;">post-MLP</span> steering:

### Gemma <span style="color: blue;">post-MLP</span> steering

<div style="display: flex; justify-content: center; margin: 2rem 0;">
  <img src="{{ '/assets/img/pre_mlp_blog/gemma_postmlp.png' | relative_url }}" alt="Gemma post-MLP" style="width:100%; height: auto;" />
</div>

Gemma becomes steerable again, particularly at token 2 and token 4 in the middle layers. 

Note: these are different models, so the activation function isn't the only variable. But the <span style="color: blue;">post-MLP</span> control experiment strongly suggests the bottleneck is inside the MLP.

### A closer look: where does the signal die?

<span style="color: red;">Pre-MLP</span> steering has to propagate through the MLP's nonlinearity, while <span style="color: blue;">post-MLP</span> steering bypasses it entirely. From [our paper](https://arxiv.org/abs/2603.00425), the output shift caused by <span style="color: red;">pre-MLP</span> steering is:

$$
\Delta \text{GLU}_{\text{steer}}(h) = 
W_d\Big[
\underbrace{(\phi'(a_g)\odot a_u)\odot (W_g \Delta h)}_{\text{gated path}}
+
\underbrace{\phi(a_g)\odot (W_u \Delta h)}_{\text{un-gated path}}
\Big] 
\\
+ O(\|\Delta h\|^2)
$$

Given a fixed steering vetor $$\Delta h$$, the output shift is modulated by two input-dependent terms:
1. $$\phi'(a_g) \odot a_u$$ in the gated path
2. $$\phi(a_g)$$ in the un-gated path

where $$a_g = W_g h$$, $$a_u = W_u h$$, and $$\phi$$ is the activation function. Our initial hypothesis was that GELU's sharper saturation (compared to SiLU) suppresses these modulation terms, killing the steering signal. However, when we plot the distributions of these terms for both models, it disproves this hypothesis:

<div style="display: flex; justify-content: center; margin: 2rem 0;">
  <img src="{{ '/assets/img/pre_mlp_blog/phi_comparison_token2.png' | relative_url }}" alt="phi comparison" style="width:100%; height: auto;" />
</div>

The modulation terms have comparable magnitude in both models, so the activation function's effect on these terms alone doesn't explain Gemma's lack of steerability. The root cause likely involves other factors (LayerNorm behavior, weight matrix structure, or how $$W_g \Delta h$$ aligns with the active dimensions), and remains an open question. What we can say is that the bottleneck is inside the MLP: <span style="color: blue;">post-MLP</span> steering bypasses it entirely and restores steerability.


## Where Does This Leave Us?



Note: the stuffs we discuss in this blog are conclusions drawn from a relatively limited set of experiments and analysis, not as rigorous as ones we did in the paper.

## References

[1] Adila, Dyah, et al. "Weight Updates as Activation Shifts: A Principled Framework for Steering." arXiv preprint arXiv:2603.00425 (2026).

[2] Yin, Fangcong, Xi Ye, and Greg Durrett. "Lofit: Localized fine-tuning on llm representations." Advances in neural information processing systems 37 (2024): 9474-9506.

[3] Li, Kenneth, et al. "Inference-time intervention: Eliciting truthful answers from a language model." Advances in Neural Information Processing Systems 36 (2023): 41451-41530.

[4] Todd, Eric, et al. "Function vectors in large language models." arXiv preprint arXiv:2310.15213 (2023).

[5] Dherin, Benoit, et al. "Learning without training: The implicit dynamics of in-context learning." arXiv preprint arXiv:2507.16003 (2025).

[6] Sakaguchi, Keisuke, et al. "Winogrande: An adversarial winograd schema challenge at scale." Communications of the ACM 64.9 (2021): 99-106.

[7] Rimsky, Nina, et al. "Steering llama 2 via contrastive activation addition." Proceedings of the 62nd Annual Meeting of the Association for Computational Linguistics (Volume 1: Long Papers). 2024.