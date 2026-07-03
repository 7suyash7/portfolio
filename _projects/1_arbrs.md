---
layout: page
title: ArbRs
description: Rust multi-protocol arbitrage engine with local Uniswap V3 and Curve simulation.
img:
importance: 1
category: systems
---

ArbRs is a Rust-based arbitrage engine built around a very practical question: can AMM math be simulated locally, fast enough, and cleanly enough to search for profitable multi-hop routes without leaning on external RPC calls for every decision?

The project reverse-engineers Uniswap V3 and Curve swap behavior into deterministic Rust code, then normalizes different pool types behind a shared `LiquidityPool` interface. That lets the engine search across heterogeneous AMM invariants without turning the route finder into protocol-specific spaghetti.

**What it shows**

- Rust systems work around numerical logic, graph search, and protocol-specific edge cases.
- A BFS-based search path for N-hop arbitrage cycle detection.
- Local simulation of swap outcomes, fees, and profitability assumptions.
- The kind of careful implementation work that maps well to inference systems: understand the runtime, reduce external dependency, and make performance measurable.

This is part of my older crypto/DeFi proof base, but I still like it because it is fundamentally a systems project: model the domain, constrain the abstractions, and keep the fast path honest.
