---
layout: post
title: "What I Learned from My Industry Job Search as a CS PhD"
date: 2026-02-23 09:00:00 -0600
description: "Notes and lessons from navigating an industry job search as a CS PhD student."
tags: [career, industry, phd]
categories: [blog]
---


I recently wrapped up my ML PhD industry job search and experienced how stressful and overwhelming the process can be. While it's still fresh, I wanted to share what I learned along the way in hopes it might help others navigating the same journey. Note: my experience was specifically on applying for Research Scientist positions. 


I break this into two parts. **Part 1** covers the end-to-end process: the first thing I did from a "let's graduate" conversation with my advisor, getting my foot in the door, prepping for different interviews, dealing with rejection, and choosing between offers. **Part 2** covers some things I learned along the way: bouncing back after a bad interview, surviving back-to-back interviews, and how having a support system keeps me sane.

# Part 1: From 'wait, I'm graduating?' to signing an offer
Last Fall I was in the first half of my 5th year, just wrapping up an internship and was excited to get back to school to kick off some new PhD projects. Back then I was still comfortable being a PhD student where I can freely explore even the most unhinged curiosities. Suddenly my advisor brings up graduating next spring.

<div style="display: flex; justify-content: center; margin: 1.5rem 0 3rem 0;">
  <div class="tenor-gif-embed" data-postid="16491396616893958961" data-share-method="host" data-aspect-ratio="1" data-width="60%" style="width: min(100%, 55%);"><a href="https://tenor.com/view/bobawooyo-dog-confused-dog-huh-dog-meme-shocked-dog-gif-16491396616893958961">Bobawooyo Dog Confused GIF</a>from <a href="https://tenor.com/search/bobawooyo-gifs">Bobawooyo GIFs</a></div>
</div>
<script type="text/javascript" async src="https://tenor.com/embed.js"></script>

It all feels overwhelming. How do I even start? With PhD research projects still ongoing, it was very tempting to procrastinate (and I did, for a while). But I eventually realized my advisor brought up graduation for a reason. I was indeed ready for independent research and to carve my path outside of the lab.

**My "job mode on" switch didn't really activate until I took my advisor's advice to start getting organized**: I made a spreadsheet to track places I applied to (or was going to apply to). Looking back, this making it official makes a lot of sense. We ML PhD students are used to filling up tables and feeling a sense of accomplishment from it (re: experiment tables :wink:). The sheet literally looks like this:

<div style="display: flex; flex-direction: column; align-items: center; gap: 1.25rem; margin: 2rem 0;">
  <img src="{{ '/assets/img/blog/excel.png' | relative_url }}" alt="Excel screenshot" style="width: 100%; height: auto;" />
  <img src="{{ '/assets/img/blog/step_fn.png' | relative_url }}" alt="Step function figure" style="width: 55%; height: auto;" />
</div>

Okay, now we are organized, we are keeping track of our applications, great. Next, how do we actually get interview invites from these places?

### Getting interviews

To put into perspective: in the current market, just applying to the company's job portal puts us into one of possibly hundreds of fellow PhD students in the market, each with their unique research and cool things to offer. How do we stand out? 

**Look out for people that are hiring.** I kept an eye on X, LinkedIn, and AI/ML conference job expos for anyone posting "We are hiring in [blabla]" where the [blabla] was right up my alley, and I'd shoot them a message. A short message conveying I had worked on relevant stuff with a pointer to my website was usually sufficient. I was lucky that my job search timeline aligned with NeurIPS, so I had the chance to meet people at the conference. Before meeting anyone, I usually planned what to say. Nothing scripted, just enough to make sure I conveyed everything I wanted them to know about my work. I also made sure to ask them questions too: it's a two-way process. After the conference, I shot an email to folks I talked to (some even reached out before I shot them an email). And when the stars aligned, interviews followed. Sometimes they didn't, and that's okay too.

### Preparing for the interviews

<div class="tenor-gif-embed" data-postid="10457554477233998682" data-share-method="host" data-aspect-ratio="1" data-width="55%"><a href="https://tenor.com/view/studying-study-gif-10457554477233998682">Studying Meme</a>from <a href="https://tenor.com/search/studying-memes">Studying Memes</a></div> <script type="text/javascript" async src="https://tenor.com/embed.js"></script>

This is where most of the beef in this blog is gonna be. I can only speak about the interview types I've gone through — it's possible there are more out there. But generally, ordered by how often I encountered them:

- ML coding
- ML debugging
- ML concepts
- Deep Learning and LLM concepts
- Job talk
- Leetcode-style coding 
- Probabilities/Statistics

Most companies I interviewed with didn't test all of these, just a subset. Usually the recruiters or my points of contact gave me a brief overview of what the interview formats were going to be. At any given point when I was in the process with a company, I tailored my prep to their specific formats. Early on, when I was just starting to prep and waiting for interview calls, I focused more on ML coding, practiced leetcode problems for timed problem solving, and rehearsed LLM knowledge to make sure I understand even the smallest details. Next I'm gonna list the specific materials I used to prep for each topic. Shout out to my friend Harshay who recommended some of these list items when I started prepping.

**ML coding.** 
- [Karpathy's NanoGPT](https://github.com/karpathy/nanoGPT)
- Practice implementing these from scratch:
    - Transformers
    - Greedy sampling
    - Positional encodings
    - Simple MLP and backprop
    - Attention and its variants
    - KV cache

**ML debugging.** This one is somewhat tied to ML coding. Understanding the implementation usually got me better at spotting errors. For extra exercise i usually ask Claude (my chosen LLM provider) to give me common implementation bugs and I try to spot them.

**Classical ML concepts**
- Brush up linear algebra basics (i watch 3b1b videos) and how the intuitions connect to ML concepts
- For list of ML topics to read/brush up, i refer to [https://www.cs.cmu.edu/~hchai2/courses/10701/](https://www.cs.cmu.edu/~hchai2/courses/10701/)

**Deep Learning and LLM concepts**
- Make sure to know:
    - pre-training: scaling laws, chinchilla compute optimal
    - post-training: RLHF, SFT, DPO, GRPO, etc
    - efficiency: flash-attention, GQA, MQA
- I refer to [https://deeplearning.cs.cmu.edu/S25/](https://deeplearning.cs.cmu.edu/S25/) and [https://pages.cs.wisc.edu/~fredsala/cs639/schedule.html](https://pages.cs.wisc.edu/~fredsala/cs639/schedule.html) for list of topics

Note: This field is very big and it can feel overwhelming to feel we *have* to know everything there is to know. I like to think the positions that are a good fit for me will test deeply on the topics related to my previous work. For example, I worked a lot on post-training, so I put an emphasis on deep knowledge in post-training (but also made sure I knew reasonably well about pre-training, efficiency, etc.)

**Job talk**
I personally find this is a fun one to prep. I assume if you made it this far, you have presented your qualifying/preliminary exam before. It is very similar to that. Altho, what I usually try to keep in mind while making my slides and practicing my talk:
- Do **not**