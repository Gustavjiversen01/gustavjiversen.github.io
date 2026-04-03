---
title: "Two Agents, More Tokens, Better Code"
description: "How pairing Claude Code and Codex in an automated review loop applies test-time compute scaling to produce better code than either model alone."
publishDate: 2026-03-20
tags: ["AI Agents", "Claude Code", "OpenAI Codex", "ML"]
draft: false
---

There is a simple observation in ML research that has become hard to ignore: spending more compute at inference time makes models produce better outputs. The concept is called test-time compute scaling, and it shows up everywhere from chain-of-thought prompting to repeated sampling with verification. The core idea is straightforward. If you give a model more tokens to think through a problem, the quality of its answer goes up. Not always linearly, not always predictably, but consistently enough that it has become a real lever for improving results.

I have been building [Claudex](https://github.com/Gustavjiversen01/claudex), a dual-agent code orchestrator that pairs Claude Code and OpenAI Codex in a structured review loop. The project started as a way to automate a workflow I was already doing manually: have one agent write a plan, have another agent tear it apart, revise, repeat. But as I built it, I realized the system is fundamentally an application of test-time compute scaling. Two agents working the same problem means roughly double the tokens applied to it, and the iterative loop multiplies that further.

## Why a second agent changes the math

When a single agent plans something and then reviews its own work, it tends to have blind spots. It made the plan. It has internalized its own assumptions. Asking it to critique itself is like asking someone to proofread their own essay right after writing it. They will miss things.

A second agent brings genuinely fresh context. It has not seen the intermediate reasoning that led to the plan. It does not share the same failure modes. When Codex reviews a plan that Claude Code generated (or vice versa), the review applies adversarial pressure that self-review cannot replicate. Different training data, different architectural biases, different tendencies. That diversity is the point.

But there is also a pure compute argument. Consider what happens in a three-iteration plan review loop. The planner generates the initial plan (one full inference pass). The reviewer analyzes it and produces severity-ranked issues (second pass). The planner revises to address those issues (third pass). The reviewer verifies the fixes and checks for regressions (fourth pass). The planner revises again (fifth pass). The reviewer does a final check (sixth pass). That is six full inference passes applied to the same planning problem. Compare that to a single agent generating a plan in one shot. The multi-agent loop spends roughly 6x the token budget, and each pass adds new information that the previous passes did not have.

This is test-time compute scaling in practice. Not through a single model thinking longer, but through multiple models iterating on each other's outputs.

## The orchestrator must be deterministic

Here is where things get interesting from an engineering perspective. If you let LLMs control the loop, you lose the scaling benefits. An LLM that decides "we are done" after one iteration because it thinks the plan looks good enough has just short-circuited the entire compute budget you intended to spend.

In Claudex, Python owns all control flow. The `FusionController` manages loop state, and the `IssueTracker` independently computes severity counts from the issue ledger. When the reviewer returns findings, the orchestrator does not ask the reviewer "is this good enough?" It counts. Critical issues, high severity issues, medium and low combined. It checks those counts against configured thresholds:

```yaml
thresholds:
  critical: 0
  high: 0
  medium_low: 4
  block_on_regressions: true
```

The `meets_threshold` function is pure Python arithmetic. No probability involved. The convergence detection uses weighted scoring across iteration snapshots (Critical=16, High=8, Medium=2, Low=1) and requires at least three data points before declaring that progress has stalled. Regression tracking flags previously resolved issues that reappear, automatically blocking the quality gate. These are operations that should never be probabilistic. Counting is not a language task. Comparing two integers is not a language task. Detecting that a score trend has flattened across three iterations is not a language task. Delegating these to an LLM introduces failure modes that have no business existing.

This is the key design principle: LLMs generate content (plans, reviews, code). Deterministic code handles all decisions about whether that content is good enough.

## What actually improves

After building and using this system, the practical improvements fall into a few categories.

**Architectural issues.** A planner agent working alone will sometimes propose a structure that works but has subtle problems: tight coupling between components, missing abstraction boundaries, inconsistent error handling patterns. A reviewer agent catches these because it is specifically prompted to challenge architectural decisions without the sunk-cost bias of having designed them.

**API and integration mistakes.** When a plan involves calling external APIs or using library features, the planner occasionally gets details wrong. Deprecated methods, incorrect parameter types, missing error cases. The reviewer, approaching the plan fresh, is more likely to flag these because it is reading the plan critically rather than recalling its own reasoning.

**Missing edge cases.** This is the most consistent improvement. Plans generated in a single pass tend to cover the happy path thoroughly but underspecify error handling, boundary conditions, and failure recovery. The adversarial review process surfaces these gaps reliably.

**Scope creep.** During implementation, Claudex reviews each phase against the approved plan using tree-object diffs. A scope enforcement setting (warn, strict, or ignore) flags files that were modified outside the plan's stated scope. This catches the common failure mode where an agent "helpfully" refactors something adjacent to the task.

## The cost trade-off

More tokens means more money. A three-iteration review loop on a complex plan can cost several dollars in API calls across both providers. That is meaningfully more than a single-pass approach.

The honest answer is that this trade-off is worth it for some tasks and not others. For a complex feature that touches authentication, database schema, and API contracts, spending an extra few dollars on thorough planning review is trivially justified. The bugs caught in review would cost hours to debug later. For renaming a variable, it would be absurd.

Claudex makes this configurable. You can set `max_iterations: 1` for quick tasks or `max_iterations: 5` with strict thresholds for critical work. The cost scales with the compute budget you choose to allocate, which is the whole point of test-time compute scaling: you get to decide how much inference to spend based on how much the outcome matters.

## Where this leads

Multi-agent systems are a natural extension of the inference scaling paradigm. Instead of making a single model think harder (chain-of-thought, tree-of-thought, repeated sampling), you distribute the compute across specialized agents with different roles and different perspectives. The planning agent and the reviewing agent do not need to be the same model, or even from the same provider. Diversity in the agent pool is a feature, not a limitation.

The more general insight is that orchestration quality determines whether additional compute translates into better outcomes. Spending 6x the tokens on a problem only helps if the loop structure ensures each pass adds real information. A reviewer that rubber-stamps everything wastes the budget. A revision step that ignores findings wastes the budget. The deterministic orchestration layer, the thresholds and convergence checks and regression tracking, exists to make sure every token spent actually contributes to the result.

If you are interested in the implementation, [Claudex is on GitHub](https://github.com/Gustavjiversen01/claudex). Two agents, one loop, deterministic quality gates. The tokens do the rest.
