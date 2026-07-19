---
layout: post
title: "AI Agents OS: Rethinking the Desktop as an AI Agent Operating System"
date: 2026-07-19
categories: research
---

## Why

After reading the paper [*AI Agents Operating System: A Heterogeneous Multi-Agent Architecture with Spontaneous Scheduling*](https://ruofeijiang.github.io/research/2026/06/28/aaos-overview.html), I decided to deeply transform an Ubuntu 26.04 LTS desktop into an AI Agent Operating System.

The paper proposes a formal definition: **AI Agents OS = (A_c, A_e, S, C)**

This is not another "ChatGPT wrapper" project. It's a fundamental architectural rethink — **LLMs are knowledge bases, not agents. The core value of an agent lies in its architecture, not its model.**

## Four Components, Each with a Clear Role

### Core Agent (A_c) — The Scheduling Core

Core Agent implements the paper's central concept: **spontaneous scheduling `schedule(t, Γ)`**.

Unlike SOP orchestration (fixed pipeline: step 1 → step 2 → step 3), spontaneous scheduling produces different plans depending on context Γ. The same task, different context, different schedule: `schedule(t, Γ₁) ≠ schedule(t, Γ₂)`.

This means:
- On failure, the system doesn't return an error code and halt — it **dynamically reschedules**
- Scheduling plans are not predefined — they're generated in real-time based on current environment
- Context includes: user preferences, system resources, agent capabilities, task history

### Execution Agent (A_e) — Model-Driven Execution

Here's the design I like most: **model capabilities determine agent roles**.

When a new model is released, the flow is:

```
New model appears → identify its strengths → define where it should be used → wrap as a new Execution Agent
```

For example:
- Strong at code → CodeAgent
- Strong at reasoning → ReasoningAgent
- Strong at vision → VisionAgent

Agent types are not fixed — they **grow naturally** with the model ecosystem. Non-overlapping functionality is a constraint (same capability, no duplicate agents), but the number of types is open.

Each Execution Agent runs sandboxed (Linux namespaces + AppArmor + cgroup), created on demand, destroyed after task completion.

### Sentinel (S) — Safety Guard for AI Agents Only

Sentinel has a clear boundary: **it only governs AI Agents, not user operations**.

- AI Agent execution → Sentinel's three-layer audit (L1 rule engine <1ms → L2 rule classifier ~5ms → L3 API audit ~200ms)
- User's own operations → Sentinel only logs, never interferes

This is not a degradation-mode special case — it's a **universal principle**. Users are responsible for their own actions. Sentinel is responsible for AI Agents only.

### CLI Integration Layer (C) — The OS↔App Boundary

The CLI layer defines an OS→App ecosystem standard, complementary to MCP's Model→Tool direction.

It also serves a critical role: **the lifeline during degradation**.

## Degradation Strategy: Fall Back to Human Operation

When the network goes down, the system doesn't crash. It doesn't fall back to a local small model either. Instead, it **gracefully hands control back to the user**:

```
Normal mode:  "Book a flight" → AI handles everything automatically
Degraded mode: "Book a flight" → AI opens the browser to the booking site → you do it yourself
```

Degradation is not "AI gets dumber" — it's "AI steps back to an assistant role, opens the tools for you, and you operate them."

Why not use a local model as fallback? Local inference means running quantized models that are significantly weaker than API flagships, while eating RAM that the desktop needs. It gives users the illusion that "AI is still working" when it's really not. Better to be honest: tell the user AI is unavailable, and help them get to the right tools.

The CLI Integration Layer naturally supports this — it doesn't need the network, only the local application registry.

## Why Pure API Is the Right Choice

Local GPU inference is often pitched as the "serious" option. The reality: you need expensive hardware to run quantized models that are still weaker than what's available via API. And your model is frozen at the version you downloaded, while API models get updated continuously.

Pure API = **low cost + strongest models + always latest + zero maintenance**.

API pricing is an early-stage phenomenon. The long-term trend is clear: inference costs drop continuously. This is not an architectural concern.

## Tech Stack

| Component | Language | Runtime |
|-----------|----------|---------|
| Core Agent | Rust | systemd system service |
| Execution Agent | Rust | transient systemd service (sandboxed) |
| Sentinel | Rust | systemd system service (independent user) |
| CLI Layer | Rust | user CLI tool |
| GNOME integration | JS | Shell Extension |
| State storage | SQLite + ZFS | filesystem (ZFS snapshots for rollback) |

Rust isn't for showboating — daemons need memory safety and low resource overhead. On a 13GB RAM machine, daemons at <10MB is the right target.

Deeply leveraging existing Linux infrastructure: systemd for service management, ZFS for state snapshots and rollback, AppArmor for sandbox isolation, D-Bus for IPC, cgroup for resource limits. No reinventing wheels.

## Architecture Assessment

### Seven Strengths

1. **Model-driven growth** — The system evolves with the model ecosystem. Faster model iteration → faster capability growth.
2. **Pragmatic degradation** — Network down ≠ system down. Fall back to human operation + CLI assistance.
3. **Clear Sentinel boundary** — Only governs AI Agents. User operations are always log-only, never interfered with.
4. **Heterogeneous non-overlap** — Same capability, no duplicate agents. Each agent is a "different species."
5. **Defense in depth** — Multi-layer sandbox + three-layer audit + ZFS immutable logs.
6. **Deep Linux integration** — All mature components, no NIH syndrome.
7. **Theory-grounded** — Based on a formal definition from the paper, not ad-hoc.

### One Open Problem

**Limited degradation intent parsing** — The local intent parser can only match known patterns ("book a flight" → open browser). For novel tasks, degradation provides no assistance. This needs solving.

### Six Clarified Non-Defects

During evaluation, we listed 7 risks. After discussion, 6 were clarified as non-architectural-defects:

| Originally listed as risk | Why it's not a defect |
|--------------------------|----------------------|
| API single point | Local inference is high-cost + low-capability; pure API is the right choice |
| No safety early on | Early stage is GitHub private repo, not production |
| Model capability boundary ambiguity | Agents are dynamically generated by spontaneous scheduling, not picked from a predefined registry |
| Scheduling complexity growth | Same as above — no need to pre-register all agents |
| Difficult rollback | Building a new distro based on Ubuntu, no need to "go back" |
| Sentinel L3 depends on network | When offline, user operates manually, Sentinel only logs — no L3 fail-closed scenario |

## Status

Plan approved. Preparing Phase 0 (Rust toolchain + system foundations).

This is not a weekend project, but every step has clear verification points. ZFS snapshots ensure each phase is rollback-safe. Six phases, incremental delivery.

---

*Based on Ruofei Jiang's paper [AI Agents Operating System](https://ruofeijiang.github.io/research/2026/06/28/aaos-overview.html).*
