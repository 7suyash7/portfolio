---
layout: post
title: LoRA Fine-Tuning a Tiny Chat Model with Unsloth
description: A learning note from an end-to-end LoRA fine-tuning exercise with Unsloth, TRL, PEFT, and a 4-bit Qwen2.5-0.5B chat model.
date: 2026-07-08
categories: [learning-notes]
tags: [llm-fine-tuning, lora, unsloth, qwen, ai-systems]
featured: true
published: true
related_posts: false
---

I recently worked through a small end-to-end fine-tuning project using LoRA, Unsloth, TRL, and a 4-bit Qwen2.5-0.5B chat model.

The goal was not to produce a powerful model. The point was to understand the full supervised fine-tuning pipeline from the inside: loading a quantized base model, attaching LoRA adapters, formatting a tiny instruction dataset, running a short SFT training job, and generating a response from the tuned model.

This was useful because it connected a few concepts that can feel abstract when reading about LLM fine-tuning. I got to see how 4-bit quantization reduces memory usage by storing large model weights in a compressed representation, while LoRA keeps the base model frozen and trains only small adapter matrices inside selected projection layers such as `q_proj`, `k_proj`, `v_proj`, and `o_proj`.

It also made the data side feel less mysterious. The model does not train on dictionaries or "prompts" directly. Each instruction/response pair has to be formatted into a consistent text template, tokenized into integer IDs, wrapped in a dataset, and passed into an SFT trainer. Seeing that flow end to end made training feel less like magic and more like a sequence of explicit engineering choices.

## What I Built

- Loaded a 4-bit quantized Qwen2.5 chat model with Unsloth.
- Verified that the model was using bitsandbytes 4-bit linear layers.
- Added LoRA adapters to attention projection modules.
- Counted total vs trainable parameters to understand the savings from PEFT.
- Built and formatted a tiny instruction dataset.
- Tokenized examples and checked sequence lengths.
- Configured a lightweight SFT training run.
- Switched the model into inference mode and generated a deterministic reply.

## What Clicked

The biggest takeaway was that fine-tuning is not just "call `train()` on a model." It is a sequence of careful decisions around model loading, quantization, adapter placement, tokenizer behavior, dataset formatting, training configuration, and inference setup.

Even though the training run was intentionally tiny, the project gave me a clearer mental model for how modern LLM fine-tuning pipelines are put together. It also gave me another small bridge from systems engineering into AI infrastructure: not just using model APIs, but understanding the machinery around memory, parameters, data formatting, training loops, and inference behavior.

Repo: [7suyash7/lora-fine-tune-a-tiny-chat-model-with-unsloth](https://github.com/7suyash7/lora-fine-tune-a-tiny-chat-model-with-unsloth)
