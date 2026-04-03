---
title: "Claudex: Two Agents, One Loop, Better Code"
description: "Automated dual-agent code quality orchestrator. One agent plans and implements, the other reviews and challenges assumptions. Python owns the control flow (thresholds, convergence, regression tracking) so no quality-gate logic is ever delegated to an LLM."
repo: "https://github.com/Gustavjiversen01/claudex"
tags: ["Python", "Claude Code", "OpenAI Codex", "AI Agents", "Pydantic", "Textual TUI"]
featured: true
publishDate: 2026-03-15
---

Claudex turns a manual two-agent workflow into a repeatable tool. One agent plans and implements. The other reviews, challenges assumptions, and verifies fixes. Python owns the control flow: loop state, thresholds, convergence checks, checkpointing, and operator controls.

## The workflow

1. Generate a structured implementation plan
2. Review that plan with a second agent
3. Feed findings back into revision until the plan meets the configured stop policy
4. Implement one phase at a time
5. Review each phase before moving on

That process works well manually, but it is slow, fragile, and repetitive. Claudex automates the orchestration while keeping the human in control through configurable quality gates, manual start/stop, and interactive triage.

## Key principle

LLMs generate plans and reviews. Python handles all control flow: counting, thresholds, convergence detection, and checkpointing. The orchestrator never trusts an LLM's self-reported issue counts.

## Three operational modes

- **tmux mode** (`claudex start`): Subprocess-driven fusion with real-time 3-pane display. Day-to-day use.
- **Dashboard mode** (`claudex start --no-tmux`): Textual TUI with inline agent panes, interactive triage, and session resume.
- **Headless mode** (`claudex plan`): Pure stdout for CI pipelines, scripts, and unattended runs.

## How quality control works

The issue tracker uses deterministic canonical IDs (`sha256(step_id:category:title)[:8]`), fuzzy Levenshtein matching for cross-iteration carry-forward, weighted convergence scoring, and regression detection. Configurable stop policies range from strict (`no_findings`) to threshold-based with severity counts and regression blocking.

## Phase-by-phase implementation review

After plan approval, implementation proceeds one phase at a time. Git tree-object baselines are stored as hidden refs (`refs/ccf/<session>/phase-N`), enabling precise per-phase diffs without polluting commit history. A scope enforcement setting (warn/strict/ignore) flags out-of-scope file changes.

## Technical details

- Configurable role swapping: assign planner, reviewer, implementer, and impl_reviewer independently to either Claude or Codex
- Async streaming runtime with concurrent stdout/stderr draining and JSONL parsing
- Session persistence with full artifact checkpointing and cross-restart resume
- Template-based prompt engineering with automatic file-reference/inline fallback at 80K characters
- Deterministic plan diffing for verification review prompts
- Environment health checks via `claudex doctor`
- 433+ tests with pytest
