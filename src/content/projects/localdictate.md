---
title: "localdictate"
description: "A local, privacy-first voice-to-text dictation tool for the desktop. Press a hotkey, speak, and the transcribed text is typed directly into the focused application. Runs entirely on-device using OpenAI's Whisper models. No cloud, no accounts, no telemetry."
repo: "https://github.com/Gustavjiversen01/localdictate"
tags: ["Python", "Whisper", "PySide6", "faster-whisper", "Desktop App"]
featured: true
publishDate: 2026-04-01
---

localdictate is a desktop dictation tool that runs entirely on your machine. It lives in the system tray, listens for a global hotkey, records your voice, transcribes it locally with Whisper, and types the result into whatever application has focus.

## How it works

1. Press `Ctrl+Space` (configurable) to start recording
2. Speak naturally
3. Press the hotkey again to stop
4. Transcribed text is automatically typed at the cursor

The entire pipeline runs locally. After the initial one-time model download from HuggingFace, no network calls are made.

## Technical design

The application is built with PySide6 (Qt 6) as a tray-only app. Audio is captured at 16 kHz via `sounddevice` and PortAudio, accumulated in a deque, then passed to `faster-whisper` (a CTranslate2-backed Whisper implementation with int8 quantization) for transcription in a background thread. Text injection uses `xdotool` on Linux and `pynput` on Windows/macOS.

## Key features

- Five quality tiers from Fast (~336 MB) to Maximum (~3 GB), with models auto-downloaded and cached
- System tray integration with visual recording/processing states
- Settings UI for model selection, microphone device, hotkey configuration, and launch-at-login
- Custom model support via any faster-whisper-compatible HuggingFace model ID
- Zero-config default. Works out of the box with the system microphone
