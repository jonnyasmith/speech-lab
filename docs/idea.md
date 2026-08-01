# Project Specification: VoiceCal - Automatic Speech Pipeline Optimizer for Linux

## Executive Summary

VoiceCal is an open-source Linux utility that automatically discovers the optimal speech recognition pipeline for a user's specific hardware, environment, and workflow.

Rather than relying on generic defaults, VoiceCal empirically measures speech recognition accuracy, latency, CPU usage, and end-to-end responsiveness by repeatedly testing different combinations of audio processing, PipeWire routing, and speech-to-text models against a known transcript.

The result is a fully tuned voice input system that works reliably for AI coding agents, terminals, editors, browsers, and desktop applications.

---

# Problem Statement

Linux has excellent speech recognition components, but no integrated system for automatically configuring them.

Users must manually experiment with:

- microphone gain
- PipeWire routing
- noise suppression
- echo cancellation
- sample rates
- model selection
- transcription engines
- latency tuning

This is time-consuming and largely guesswork.

VoiceCal replaces guesswork with automated benchmarking.

---

# Primary Goals

1. Automatically determine the best microphone processing pipeline.
2. Benchmark multiple local speech recognition engines.
3. Optimize for both accuracy and responsiveness.
4. Produce a reusable configuration.
5. Integrate cleanly with modern Linux desktops.
6. Be completely offline by default.

---

# Target Platform

Primary target:

- Fedora
- COSMIC Desktop
- PipeWire
- WirePlumber
- Wayland

Secondary support:

- GNOME
- KDE Plasma
- Sway
- Hyprland

---

# Target User

Developers using:

- AI coding agents
- terminal workflows
- local LLMs
- voice assistants
- transcription
- dictation

Especially users running tools such as:

- OMP
- Aider
- Claude Code
- Codex CLI
- OpenCode
- custom AI harnesses

---

# High-Level Architecture

VoiceCal consists of several independent modules.

## Module 1 — Calibration

Purpose:

Collect known speech samples.

The user reads predefined passages designed to cover:

- normal conversation
- technical vocabulary
- programming terms
- filenames
- punctuation
- numbers
- symbols
- command line syntax

Example:

```
Create a new Python virtual environment.

Run:

python -m venv .venv

Then activate it.

The function accepts two parameters:
cache underscore directory
and max underscore retries.
```

The expected transcript is already known.

This becomes the ground truth.

---

## Module 2 — Recording Engine

Uses PipeWire.

Responsibilities:

- microphone selection
- repeatable recordings
- timestamping
- metadata collection

Configurable:

- sample rate
- channel count
- bit depth
- buffer size

---

## Module 3 — Audio Processing Pipeline

Generate many processing chains automatically.

Possible processing nodes:

None

Noise suppression

WebRTC noise suppression

RNNoise

Echo cancellation

Automatic gain control

Compression

Limiter

EQ

High-pass filter

Low-pass filter

Normalization

Silence trimming

Gain adjustment

Microphone boost

Each processing chain becomes a candidate.

Example:

```
Mic

↓

Noise suppression

↓

Gain +6 dB

↓

High-pass

↓

Recorder
```

---

## Module 4 — Speech Recognition Benchmark

Support multiple engines.

Examples:

faster-whisper

whisper.cpp

OpenAI Whisper

Moonshine

Sherpa-ONNX

Vosk

NVIDIA Parakeet

Future engines should be pluggable.

For each engine test:

recording

↓

transcription

↓

metrics

Metrics:

Word Error Rate

Character Error Rate

Token accuracy

Latency

Throughput

CPU usage

RAM usage

GPU usage

Energy consumption (optional)

---

## Module 5 — Optimizer

Instead of brute force, intelligently search configuration space.

Possible algorithms:

Random Search

Bayesian Optimization

Genetic Algorithms

Tree Parzen Estimator

Hill Climbing

Objective function:

maximize:

accuracy

minimize:

latency

CPU

RAM

Composite score example:

```
score =
0.50 × accuracy
+
0.30 × latency
+
0.10 × cpu
+
0.10 × memory
```

Weights configurable.

---

## Module 6 — AI-Assisted Optimizer

Optional.

Allow an LLM to analyse benchmark results.

Example tasks:

"This microphone consistently clips."

"RNNoise improves accuracy by 11%."

"Sample rates above 24kHz provide no benefit."

Suggest new experiments dynamically.

---

## Module 7 — End-to-End Assistant Benchmark

Instead of only benchmarking speech recognition:

Measure:

Hotkey pressed

↓

Recording starts

↓

Recording stops

↓

Speech recognition

↓

Prompt injected

↓

LLM starts generating

↓

First token

↓

TTS starts

↓

Speech ends

Collect:

time to transcription

time to first token

time to first spoken response

overall conversation latency

This is arguably the most useful benchmark.

---

# PipeWire Integration

Treat PipeWire as a graph.

Allow experimentation with routing.

Example:

Mic

↓

Noise suppression

↓

Branch

↓

Whisper

↓

Discord

↓

Recorder

Simultaneously:

LLM

↓

Piper

↓

Speakers

Avoid feedback loops.

---

# Benchmark Report

Generate rich reports.

Example:

```
Model:
faster-whisper base.en

Accuracy:
98.3%

Latency:
410 ms

CPU:
42%

RAM:
820 MB

Composite:
94.1
```

Graphs:

accuracy vs latency

model comparison

CPU comparison

pipeline comparison

confidence histograms

---

# Configuration Output

Automatically generate:

PipeWire configs

WirePlumber configs

VoiceCal config

Whisper config

Hotkey config

Suggested environment variables

Example:

```
~/.config/voicecal/

pipeline.toml

voice.conf

models.toml
```

---

# Runtime Mode

After calibration:

One hotkey.

Workflow:

Press

↓

record

↓

release

↓

transcribe

↓

paste into focused application

↓

(optional)
auto-submit

Target applications:

terminal

browser

editor

IDE

AI chat

No application-specific integration required.

---

# Optional Features

Wake word

Voice activity detection

Continuous dictation

Command mode

Macro mode

Voice shortcuts

Clipboard mode

Remote microphone (phone)

Bluetooth microphone support

USB microphone comparison

---

# Remote Device Support

Future feature.

Allow an iPhone or Android phone to become the microphone.

Workflow:

Phone

↓

Wi-Fi

↓

PipeWire virtual source

↓

Whisper

↓

Focused application

Benefit:

Use Apple's excellent microphones and dictation-quality audio capture while keeping all speech recognition local.

---

# Plugin Architecture

Everything should be replaceable.

Plugins:

Audio processors

STT engines

Optimizers

Report generators

LLM analysers

Exporters

TTS engines

---

# Technology Stack

Language:

Python (rapid development)

Performance-critical modules:

Rust

Libraries:

PipeWire

WirePlumber

faster-whisper

ONNX Runtime

PyTorch (optional)

Rich

Typer

Pydantic

SQLite

Matplotlib

---

# Success Metrics

The project is successful if it can:

- reduce transcription error compared to default settings
- identify the best local model automatically
- produce a reproducible configuration
- reduce end-to-end AI interaction latency
- work with any focused application
- require minimal manual tuning

---

# Long-Term Vision

VoiceCal becomes for Linux speech recognition what benchmark tools are for GPUs.

Instead of users asking:

"Which Whisper model should I use?"

they simply run:

```
voicecal calibrate
```

Ten minutes later they receive:

- the best audio pipeline
- the best speech model
- the fastest configuration
- the lowest latency
- automatically generated configuration files
- a benchmark report explaining every decision

The system should transform Linux speech recognition from a manually tuned process into a measurable, repeatable, and self-optimizing workflow.

## Future Stretch Goal

Expand beyond speech recognition into a complete AI interaction benchmark suite that optimizes:

- microphone pipeline
- speech recognition
- prompt injection
- LLM response latency
- text-to-speech
- full conversational round-trip time

This would allow users to optimize the complete voice assistant experience rather than individual components.
