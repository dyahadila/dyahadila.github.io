---
layout: post
title: "The Case for Pre-MLP Steering"
date: 2026-03-05 09:00:00 -0600
description: "interesting stufs we learned about pre-MLP steering"
tags: [research]
categories: [blog]
---

Recently, we published [our paper](https://arxiv.org/abs/2603.00425) on principled activation steering (if you haven't, I recommend checking out the paper! it is the PhD work I am most proud of :wink:). We found that post-block steering — after the residual connection is added back to the MLP output — is the theoretically backed, most expressive steering site. If your goal is replicating weight-based fine-tuning with orders or magnitude less parameters, post-block is the way to go.

But that doesn't mean pre-MLP steering isn't interesting. During our analysis, we discovered that pre-MLP steering (while less expressive for fine-tuning-like effects) reveals something interesting about how models process context. In fact, it has a neat connection to in-context learning. In this blog we are going to discuss interesting things we learned about pre-MLP steering during the project. Mainly:

1. Pre-MLP steering has a strong connection with in-context learning (ICL).
2. The choice of a model's *activation function* plays a role into how steerable a model is when we do pre-MLP steering


## Pre-MLP steering formulation

First let's set up some notation and define what we mean by pre-MLP steering. Following notation in our paper, steering updates a model's activations as follows:
$$ h \to h + \delta h $$. Pre-MLP steering does this update on activations before the MLP. In our experiments we steer the output of attention (red text arrow below), but there are works that steer specific attention heads like those in [1, 2, 3].

<div style="display: flex; justify-content: center; margin: 2rem 0;">
  <img src="{{ '/assets/img/pre_mlp_blog/steering_sites.png' | relative_url }}" alt="Jintervention sites" style="width:60%; height: auto;" />
</div>

## In Context-Learning and Pre-MLP Steering are closely related

TLDR: *It turns out that what ICL does implicitly — shifting attention outputs based on context — is what pre-MLP steering does explicitly*

## Final Remarks

Note that the stuffs we discuss in this blog are conclusions drawn from a relatively limited set of experiments and analysis, not as rigorous as ones we did in the paper.

## References

[1] Yin, Fangcong, Xi Ye, and Greg Durrett. "Lofit: Localized fine-tuning on llm representations." Advances in neural information processing systems 37 (2024): 9474-9506.

[2] Li, Kenneth, et al. "Inference-time intervention: Eliciting truthful answers from a language model." Advances in Neural Information Processing Systems 36 (2023): 41451-41530.

[3] Todd, Eric, et al. "Function vectors in large language models." arXiv preprint arXiv:2310.15213 (2023).

