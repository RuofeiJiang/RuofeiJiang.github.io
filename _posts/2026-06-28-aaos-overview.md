---
layout: post
title: "AAOS: Rethinking AI Agents as an Operating System"
date: 2026-06-28
categories: research
---

## What is an AI Agent System?

The term "AI Agent System" is everywhere—yet no one has formally defined it. Existing frameworks treat it as a descriptive phrase, not a rigorous concept. This gap matters: without a definition, we cannot distinguish an AI *system* from an AI *operating system*, nor can we reason about what architectures are possible.

In my paper [*AI Agents Operating System: A Heterogeneous Multi-Agent Architecture with Spontaneous Scheduling*](https://github.com/RuofeiJiang/ColoMind/blob/main/docs/paper-aaos.md), I propose **AAOS (AI Agents Operating System)**—a formal definition of AI agent systems and an architecture that follows from that definition.

## The Core Argument: LLMs Are Knowledge Bases, Not Agents

Current frameworks conflate LLMs with agents. This is the root error. An LLM can *know* and *reason*, but it cannot *do*—it cannot make autonomous decisions, schedule resources, or operate external systems. These capabilities come from **architecture**, not from the model.

This view aligns with cognitive architecture research: both SOAR and ACT-R model agents as the combination of an architecture (providing decision-making, scheduling, memory management) and a knowledge base (providing domain knowledge). The knowledge base alone is not the agent.

**Implication**: The core value of an agent lies in its architecture, not its model. Models are replaceable, composable, upgradable—architecture is what makes an agent an agent.

## Three-Layer Heterogeneous Agent Architecture

AAOS is defined as a four-tuple: **AAOS = (A_c, A_e, S, C)**

| Component | Role | Lifecycle | Trust Level |
|-----------|------|-----------|-------------|
| **Core Agent** (A_c) | Scheduling core: analyze, schedule, integrate, orchestrate | Permanent | Trusted |
| **Execution Agent** (A_e) | Model cluster wrappers: generate, vision, code, audio, CLI | Ephemeral | Untrusted (sandboxed) |
| **Sentinel** (S) | Independent safety system: monitor, audit, takeover | Permanent | Absolute |
| **CLI Integration Layer** (C) | External capability extension | — | — |

The key constraint is **heterogeneity**: no two different agent types share overlapping functions. Core Agents schedule, Execution Agents execute, Sentinel safeguards—they are different species, not different instances of the same species.

## Spontaneous Scheduling vs. SOP Orchestration

Existing multi-agent frameworks (AutoGen, CrewAI, LangChain, MetaGPT) all rely on some form of pre-defined orchestration—SOP pipelines, chain calls, or role presets. The scheduling function is a fixed mapping: same input, same output.

AAOS proposes **spontaneous scheduling**: the Core Agent's scheduling function *schedule(t, Γ)* depends on task context Γ and LLM reasoning, not on a pre-defined mapping. For the same task, different contexts may produce different schedules: *schedule(t, Γ₁) ≠ schedule(t, Γ₂)*. This is a feature, not a bug—it enables dynamic adaptation when execution fails or context changes.

The chain of causality is clear: **LLM-as-agent → homogeneous agents → no functional division → external orchestration → SOP**. AAOS breaks this chain at the start: LLM is a knowledge base, agents are defined by architecture, therefore heterogeneous agents with spontaneous scheduling become possible.

## The CLI Integration Layer: Where System Meets Operating System

The boundary between an AI system and an AI operating system is the **CLI Integration Layer**. Without it, an agent's capabilities are limited to built-in functions. With it, external applications can plug in via a standard interface, extending the agent's capability boundary.

This mirrors how traditional operating systems work: POSIX defined standard interfaces for Unix, enabling applications to be portable across systems. AAOS defines the **AAOS-CLI Specification**—external software adapts to the system, not the other way around.

**MCP vs. AAOS-CLI**: MCP is a model-side tool-calling protocol (Model→Tool direction). AAOS-CLI is an OS-side ecosystem standard (OS→App direction). They can coexist: Execution Agents may use MCP to call tools, but the CLI Integration Layer defines how external software exposes itself to the operating system.

## Sentinel: Safety as Architecture, Not Add-on

Sentinel is an independent, permanently-running safety system with absolute trust level. It operates on two fronts:

1. **Liveness monitoring** (local): heartbeat checks, anomaly detection, takeover when the Core Agent fails
2. **I/O auditing** (neutral institution): input/output review with a three-layer defense pipeline—L1 rule engine (<1ms), L2 local model (~50ms), L3 LLM takeover (~200ms)

Critically, the I/O audit rules should be maintained by a **neutral institution**, not by the system operator. The operator has commercial incentives to weaken safety standards; a neutral institution ensures audit rules are fair and consistent.

## A Concrete Example: Business Trip Planning

Consider the request: *"Plan my business trip to Beijing next week—book flights, hotel, and organize meeting materials."*

**In AAOS:**
1. Core Agent decomposes into three sub-tasks: book flights, book hotel, organize materials
2. Spontaneous scheduling assigns: Travel Agent (flights + hotel), Document Agent (materials)—parallel execution
3. Sentinel flags payment operations as requiring user confirmation; read-only tasks auto-pass
4. If the preferred flight is sold out, Core Agent dynamically reschedules with a different airline—no interruption
5. Core Agent integrates all results, Sentinel audits output, returns to user

**In SOP orchestration:**
1. Pre-defined pipeline: search flights → book flights → search hotel → book hotel → organize materials (serial)
2. If flights are sold out: returns error code, entire pipeline halts, requires manual intervention
3. No parallelism: total time = t₁ + t₂ + t₃
4. Safety checks are global interceptors—cannot distinguish payment operations from information queries

## Key Contributions

1. **Formal definition** of AI agent systems, distinguishing AI systems from AI operating systems
2. **LLM as knowledge base**: agents are defined by architecture, not by models
3. **Three-layer heterogeneous architecture**: Core Agent, Execution Agent, Sentinel—functionally non-overlapping
4. **Spontaneous scheduling paradigm**: context-dependent scheduling vs. SOP passive orchestration
5. **CLI Integration Layer**: the dividing line between system and operating system; "applications adapt to the system" as the ecosystem direction
6. **Sentinel neutrality**: I/O auditing maintained by neutral institutions, technical independence + institutional independence

---

*This post is an overview of my paper "AI Agents Operating System: 人工智能体集群操作系统." The full manuscript has been submitted to the Journal of Artificial Intelligence Research (JAIR) and is currently under review. If you are interested in the complete paper and are not involved in the review process, please feel free to contact me directly.*
