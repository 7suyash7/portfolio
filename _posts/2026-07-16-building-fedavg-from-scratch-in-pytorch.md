---
layout: post
title: Building FedAvg From Scratch in PyTorch
description: A learning note from implementing Federated Averaging in PyTorch, with client-local training, sample-weighted aggregation, and IID vs non-IID data splits.
date: 2026-07-16
categories: [learning-notes]
tags: [federated-learning, pytorch, fedavg, distributed-training, ai-systems]
featured: true
published: true
related_posts: false
---

I recently built a small Federated Averaging implementation from scratch in PyTorch. The goal was not to chase benchmark performance, but to understand what actually happens inside FedAvg: how the server initializes a model, how clients train locally, how their weights are averaged, and how IID vs non-IID client data affects learning.

The setup was intentionally simple. I used a tiny synthetic classification dataset and a two-layer MLP classifier. The model takes an input feature vector, passes it through a hidden layer with a ReLU activation, and outputs raw logits for each class. I deliberately kept the model small because the interesting part of this project was not the architecture. The interesting part was the training system around it.

The basic pipeline looked like this:

```text
Build synthetic dataset
-> split into train/test
-> partition train data across clients
-> initialize global model
-> select clients each round
-> train selected clients locally
-> aggregate client models with sample-weighted averaging
-> evaluate global model
```

The most important thing I learned is that FedAvg is not conceptually complicated, but it is very easy to implement incorrectly in subtle ways.

In normal centralized training, there is one model and one dataset. You shuffle the data into batches, compute loss, backpropagate, and update the model. In FedAvg, every selected client gets a copy of the same global model, trains that copy on its own local data, and sends the updated weights back to the server. The server then creates a new global model by averaging those client weights.

So instead of thinking:

```text
one model trains on all data
```

FedAvg is more like:

```text
many client models start from the same global weights
each trains locally
the server averages their final parameters
```

That distinction mattered a lot. One bug I had to avoid was accidentally training multiple clients on the same model object. That would make client updates sequential instead of independent, which breaks the whole "train locally, then average" structure. Each client needs a fresh model loaded with the same global state at the start of the round.

Another thing I paid attention to was how model state is copied. PyTorch `state_dict`s are dictionaries of tensors, but blindly passing them around can create shared references. For FedAvg, the server needs safe snapshots of model parameters. That means cloning and detaching tensors so later optimizer updates do not mutate a state that was supposed to be frozen.

## Sample-Weighted Aggregation

The aggregation step was a useful systems lesson. FedAvg does not simply average clients equally unless every client has the same amount of data. The correct update is sample-weighted:

```text
new_global = sum((client_samples / total_samples) * client_state)
```

This means a client with 100 examples should influence the new global model more than a client with 10 examples. Implementing this forced me to treat a model not as one object, but as a collection of named tensors: `fc1.weight`, `fc1.bias`, `fc2.weight`, and `fc2.bias`. Aggregation is just tensor math applied key-by-key across matching state dictionaries.

## IID vs Non-IID Clients

The IID vs non-IID data split was probably the most interesting part. In the IID case, each client gets a roughly representative slice of the global dataset. In the non-IID case, I sorted examples by label, split them into shards, and assigned only a few shards to each client. That creates label-skewed clients where each client may only see a small part of the label space.

This helped make the main FedAvg problem more concrete. With IID data, clients usually produce updates that point in somewhat compatible directions. With non-IID data, one client might mostly learn class 0, another might mostly learn class 1, and another might mostly learn class 2. When the server averages these models, the result can be noisier or worse than the IID case. This is the client drift problem: local training can pull each client model toward its own biased data distribution.

I also experimented with two important knobs:

```text
local_epochs: how much each client trains before aggregation
client_fraction: what fraction of clients participate each round
```

Increasing local epochs means each client does more work per communication round. This can reduce the number of server rounds needed, but it can also make client drift worse, especially when the data is non-IID. Increasing the client fraction means the server averages more client updates per round, which usually gives a more representative global update, but costs more communication.

## Details That Mattered

The biggest implementation details I had to get right were:

```text
Use one shared permutation when shuffling features and labels.
Never apply softmax before cross-entropy loss.
Reset gradients before each optimizer step.
Clone and detach model state before storing it.
Load the same global state into every selected client.
Aggregate client states with sample-count weighting.
Use fixed seeds so experiments are reproducible.
```

The softmax point was a good reminder. The model returns raw logits, and `CrossEntropyLoss` handles the softmax/log-softmax internally. Adding softmax manually would double-normalize the outputs and damage the gradients.

The seed handling was also more important than I expected. For controlled experiments, I used the same seed across runs when comparing client fractions or local epoch counts. That way, the only thing changing was the variable being tested. Otherwise, randomness in initialization, client selection, or batch ordering could make the comparison noisy.

## What Clicked

Overall, this project helped me see FedAvg as a simple but powerful distributed training pattern:

```text
broadcast weights
train locally
average weights
repeat
```

The hard part is not the high-level idea. The hard part is preserving the right invariants: clients must start from the same global model, data-label pairs must stay aligned, state dictionaries must not accidentally share storage, and aggregation must respect how much data each client trained on.

This was a good from-scratch project because it made federated learning feel less abstract. FedAvg is basically ordinary SGD running on many local datasets, plus careful parameter averaging on the server. Once I implemented the pieces myself, the tradeoffs around IID vs non-IID data, local epochs, client sampling, and communication cost became much easier to reason about.
