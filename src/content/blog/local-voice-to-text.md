---
title: "Why Local Speech-to-Text Still Matters"
description: "Every voice dictation tool I found required a subscription. So I built one that runs Whisper locally, works on any computer, and costs nothing."
publishDate: 2026-04-02
tags: ["AI", "Whisper", "Python", "Open Source"]
draft: false
---

I wanted to dictate text on my computer. Speak into my microphone, have words appear at the cursor. Simple enough in concept. Every tool I found wanted $5, $10, or $15 a month for the privilege. Not for cloud compute on some difficult task. Just to use my own CPU to turn my own voice into text.

That felt wrong. So I built [OSW](https://github.com/Gustavjiversen01/OSW) (Open Source Whisper), a free, local voice-to-text tool that runs entirely on your machine.

But this post is not really about OSW. It is about why local speech recognition is now practical for anyone, and what it took to get here.

## The subscription trap

Speech-to-text is a solved problem. The models exist, they are open, and they run on consumer hardware. Yet the market is dominated by tools that route your audio through a cloud API and charge monthly for it. Some of them are genuinely using server-side models for quality reasons. Many are not. They are wrapping the same open-source models you could run yourself, adding a subscription layer, and hoping you do not notice.

I am not opposed to paying for software. But recurring payments for something that requires zero ongoing server cost is a pricing model designed to extract money, not reflect actual expense. Once you download a Whisper model, the only resource it consumes is your own electricity.

## What Whisper changed

In September 2022, OpenAI released Whisper as an open-source model. It was trained on 680,000 hours of multilingual audio, and it was shockingly good. Not "good for an open model." Just good. It handled accents, background noise, and casual speech better than most commercial APIs at the time.

More importantly, they released the weights. Anyone could download the model and run inference locally. That single decision shifted the entire landscape. Speech-to-text went from "cloud API you rent" to "artifact you own."

The original Whisper implementation was slow, though. Running the large model on CPU was painful. You would speak for five seconds and wait thirty for the transcription. Usable for batch processing, not for interactive dictation.

## The faster-whisper ecosystem

The breakthrough for local, real-time use came from the community. [CTranslate2](https://github.com/OpenNMT/CTranslate2) is an inference engine optimized for Transformer models. The [faster-whisper](https://github.com/SYSTRAN/faster-whisper) project used it to reimplement Whisper inference with int8 quantization, cutting memory usage and latency dramatically.

What does int8 quantization actually mean here? The original Whisper models store their weights as 32-bit floating point numbers. Quantization converts those to 8-bit integers, which are cheaper to compute and take up roughly a quarter of the memory. You lose some precision, but for speech recognition the accuracy difference is negligible. A 3 GB model fits comfortably in RAM on any modern machine.

Then came the distilled models. Researchers at Hugging Face trained smaller versions of Whisper that retain most of the accuracy at a fraction of the size. The distil-medium.en model, for example, weighs about 800 MB and handles English dictation nearly as well as the full large-v3. For most people dictating notes or emails, the difference is imperceptible.

The result: you can run high-quality speech recognition on a laptop with no GPU. Not as a proof of concept. As an everyday tool.

## Building OSW

When I sat down to build this, I had a few constraints in mind.

**It should be invisible.** No main window, no browser tab, no Electron app eating 400 MB of RAM in the background. OSW runs as a system tray icon. A small red dot appears while recording. That is the entire UI during normal use.

**It should be instant to invoke.** Press `Ctrl+Space`, speak, press it again. The text appears wherever your cursor is. The hotkey is global and configurable. The interaction model is: think, press, speak, press, keep working. No context switching.

**It should work with any application.** This was the trickiest part. On Linux, I use `xdotool` to simulate keystrokes, which types the transcribed text into whatever window has focus. On Windows and macOS, `pynput` handles the same job. The text injection looks like real keyboard input to the receiving application, so it works in terminals, text editors, chat apps, and anywhere else you can type.

The audio pipeline is straightforward. `sounddevice` captures 16 kHz mono audio (the sample rate Whisper expects) and accumulates frames in a `collections.deque`. When you stop recording, the buffer is concatenated into a NumPy array and handed to faster-whisper on a background thread. The transcription result comes back as text, gets emitted via a Qt signal to the main thread, and gets injected at the cursor. The whole thing is about 400 lines of Python across five modules.

One design decision I am happy with: lazy model loading. The model does not load at startup. It loads the first time you actually dictate something. If you switch quality tiers in settings, the old model is unloaded and garbage collected before the new one loads. This keeps idle memory usage minimal.

## Model size trade-offs

OSW ships with five quality tiers, and choosing between them is the one real decision a user has to make.

The **Fast** tier uses the distil-small.en model at about 336 MB. It transcribes almost instantly, even on older hardware. Accuracy is decent for clear speech and short phrases. It struggles with technical jargon, mumbling, and longer sentences.

The **Balanced** tier (the default) uses distil-medium.en at about 800 MB. This is the sweet spot for most people. It handles natural speech well, deals with filler words and pauses gracefully, and transcribes in roughly real-time on a modern CPU.

The **Maximum** tier uses the full large-v3 model at about 3 GB. This is the best accuracy you can get locally. It handles accents, code-switching between languages, and obscure vocabulary noticeably better. But transcription takes longer, and the model takes a few seconds to load on first use. Worth it if accuracy is critical. Overkill for quick notes.

My advice: start with Balanced. Switch to Quality or High if you find yourself correcting too many words. You will know within a few minutes of use whether you need more accuracy.

## The privacy angle

Here is something that gets overlooked in the "cloud vs. local" debate. Every cloud-based dictation service receives a recording of your voice. Even if they promise not to store it, the audio travels over the network and is processed on someone else's server. If you are dictating passwords, medical notes, legal documents, or anything you would not want a third party to hear, that matters.

With local inference, the audio never leaves your machine. There is no network call after the initial model download. No telemetry. No analytics. Your voice data exists only in RAM for the few seconds it takes to transcribe, then it is gone.

This is not a theoretical concern. It is a practical reason to prefer local tools for sensitive work.

## Where local speech recognition is heading

The trajectory is clear. Models are getting smaller and faster without losing accuracy. Hardware is getting better at running them. Two years ago, running Whisper locally was a research project. Today it is a pip install.

I expect the next wave will bring real-time streaming transcription to consumer hardware (words appearing as you speak, not after you stop), better support for speaker diarization, and tighter OS-level integration. Apple already ships local dictation on macOS and iOS. It is only a matter of time before Linux desktops and Windows offer something comparable out of the box.

Until then, tools like OSW fill the gap. A free, open, local alternative to subscription dictation. Your microphone, your CPU, your text. Nothing else involved.

If you want to try it: [github.com/Gustavjiversen01/OSW](https://github.com/Gustavjiversen01/OSW). It is MIT licensed. Contributions are welcome.
