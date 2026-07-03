---
layout: page
title: Reth P2P Mempool Crawler
description: Ethereum mainnet P2P ingestion pipeline using Reth components, Tokio, PostgreSQL, and WebSockets.
img:
importance: 2
category: systems
---

This project is a standalone Ethereum networking tool that uses Reth components to handshake with mainnet peers, ingest pending transactions from P2P gossip, and stream the resulting propagation data into storage and live interfaces.

The core architecture separates network I/O from downstream processing. Raw transaction flow enters an async Rust pipeline, moves through Tokio MPSC channels, lands in PostgreSQL for indexing, and can be streamed to WebSocket/TUI consumers for real-time observability.

**What it shows**

- Direct work with Ethereum P2P networking, devp2p/RLPx, and transaction propagation.
- Async Rust pipeline design with backpressure-aware boundaries.
- PostgreSQL indexing for live transaction and MEV analysis.
- A clean bridge from low-level networking to user-facing observability.

The AI/ML relevance is not cosmetic. Inference infrastructure also needs disciplined ingestion, queueing, backpressure, observability, and careful latency accounting. This project is one of the reasons I am drawn to that side of AI engineering.
