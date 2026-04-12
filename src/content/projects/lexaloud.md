---
title: "Lexaloud: Neural Text-to-Speech for Academic Reading"
description: "A local, privacy-first TTS daemon for Linux. Select text in any application, press a hotkey, and hear it read aloud sentence by sentence using Kokoro-82M on your own GPU. No cloud, no accounts, no telemetry."
repo: "https://github.com/Gustavjiversen01/lexaloud"
tags: ["Python", "FastAPI", "ONNX Runtime", "GTK3", "Kokoro-82M", "systemd"]
featured: true
publishDate: 2026-04-09
---

Lexaloud is a Linux text-to-speech daemon built for reading academic papers and long articles aloud. It runs a neural voice model locally on your machine, streams audio sentence by sentence with pause/skip/rewind controls, and communicates over a Unix domain socket. Nothing leaves your computer.

## How it works

1. **Select text** in any application (PDF reader, browser, terminal)
2. **Press a global hotkey** (e.g., `Ctrl+0`)
3. **Listen** as each sentence is synthesized and played back in order, with full transport controls

The CLI grabs the primary selection via `wl-paste` or `xclip`, sends it over a Unix domain socket to a FastAPI daemon running as a `systemd --user` unit, and the daemon handles preprocessing, synthesis, and playback. The GTK3 tray indicator and control window provide visual state feedback and voice/speed settings on desktops that support AppIndicator.

## Architecture

Three components, loosely coupled by a Unix domain socket:

- **FastAPI daemon** (`systemd --user`): owns the TTS provider, playback state machine, and audio sink. Binds `$XDG_RUNTIME_DIR/lexaloud/lexaloud.sock` via systemd's `RuntimeDirectory` — only the owning user's processes can reach it. No TCP port to firewall, no cross-user attack surface.
- **CLI** (`lexaloud speak-selection`, `pause`, `skip`, `back`, `stop`): captures the X11/Wayland selection and sends HTTP requests to the daemon.
- **GTK3 tray indicator + control window**: voice preview and switching across 12 built-in voices (American and British, male and female), speed control, and hotkey configuration.

## Key technical decisions

### Sentence-granularity streaming

Kokoro-82M emits a whole-sentence waveform in one call, so the natural unit of control is the sentence. A producer task synthesizes sentences in a `ThreadPoolExecutor` and pushes `AudioChunk`s to a bounded `asyncio.Queue(maxsize=3)`. A consumer task pulls chunks and writes them to the audio sink in sub-chunk blocks (~100 ms), checking pause/cancel between each block. This gives clean pause, skip, and rewind semantics without audio clipping.

### Bounded backpressure

The bounded queue between producer and consumer caps memory usage. When the user pauses, the producer blocks on `put()` after the queue fills, so arbitrarily long pauses use constant RAM regardless of document length.

### Cooperative cancellation via job IDs

Stop, skip, and back commands bump a monotonic job ID. The provider checks `is_current_job(job_id)` at key points and returns `None` for stale jobs. This avoids mid-call thread cancellation while staying robust under concurrent HTTP requests.

### ONNX Runtime GPU with silent-degradation detection

The provider calls `onnxruntime.preload_dlls(cuda=True, cudnn=True)` before session construction and verifies that `session.get_providers()` actually contains `CUDAExecutionProvider` post-build. Without this, the CUDA EP silently falls back to CPU on Ubuntu 24.04 with the pip NVIDIA wheels — a failure mode discovered during early prototyping.

### Unix domain socket, not TCP loopback

The daemon only listens on UDS. There is no port to scan, no `127.0.0.1` surface where any local process can inject speech requests, and systemd's `RuntimeDirectoryMode=0700` enforces user isolation.

## Preprocessing pipeline

Academic PDFs are messy. The preprocessor strips common PDF extraction artifacts, expands Latin abbreviations (e.g., "et al.", "i.e."), and segments text into sentences before synthesis. Sentences exceeding `MAX_SENTENCE_CHARS` are rejected at the API boundary rather than silently truncated.

## Scope and constraints

- Linux only. Primary support for Ubuntu/Debian; secondary for Fedora, Arch, Mint, Pop!_OS.
- NVIDIA GPU with CUDA 12 for real-time synthesis. CPU fallback runs at ~10x real-time, which is usable for reading along.
- Single user, single audio stream. No mixing, no session persistence across daemon restarts.
- 166 tests covering the daemon, CLI, provider, preprocessing, and playback state machine.
