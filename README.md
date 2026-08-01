# SpeechLab

An open-source Linux utility that **measures** its way to a good speech pipeline instead of
guessing at one.

Linux already has the pieces — PipeWire, RNNoise, WebRTC noise suppression, faster-whisper,
whisper.cpp, Sherpa-ONNX, Vosk. What it does not have is any way to know which combination
of them is right for *your* microphone, *your* room, and *your* machine. Today that answer
comes from forum threads and an afternoon of trial and error.

SpeechLab records you reading a passage whose transcript is already known, runs that audio
through many candidate processing chains and recognition engines, scores each one on word
error rate, latency, CPU and memory, and then writes out the configuration that won.

The end state is one command:

```
speechlab calibrate
```

and, afterwards, a hotkey that dictates into whatever window has focus — terminal, editor,
browser, or AI coding agent — with no per-application integration and no network calls.

## Status

**Pre-implementation.** There is no code in this repository yet: the design is settled, the
work is decomposed, and the first milestone has not started. Nothing here is installable.

- [docs/idea.md](docs/idea.md) — the full specification.
- [docs/roadmap.md](docs/roadmap.md) — milestones, sequencing, and the open questions that
  are still genuinely open.
- [docs/adr/](docs/adr/) — decisions and the reasoning behind them.

## What it will do

**Calibrate.** Read a passage covering conversational speech, technical vocabulary,
filenames, punctuation, numbers, and command-line syntax. That recording becomes ground
truth for everything else.

**Benchmark.** Sweep audio processing chains (noise suppression, echo cancellation, gain,
filtering, normalisation) against multiple local STT engines and model sizes. Every trial
lands in SQLite with its accuracy and resource metrics.

**Optimize.** Search the configuration space under a trial budget rather than brute-forcing
it, scoring on a composite of accuracy, latency, and resource cost.

**Emit config.** `pipeline.toml`, `voice.conf`, `models.toml`, plus PipeWire and WirePlumber
fragments — a reproducible setup, not a screenshot of settings.

**Run.** Press a hotkey, speak, release; the transcription is pasted into the focused
application, optionally auto-submitted.

Everything runs locally. Offline by default is a requirement, not a mode.

## How it gets built

The specification is written as seven modules. Building it module-by-module would produce
six finished components and nothing runnable, so the roadmap is sliced differently: **every
milestone ends in a command a user can execute.**

| Milestone | Ships |
|---|---|
| M0 | Four spikes, each answering a question that changes the architecture |
| M1 | `speechlab record` / `speechlab bench` — one pipeline, one engine, a real WER number |
| M2 | Pluggable STT interface across Whisper and non-Whisper engines |
| M3 | Dictation runtime — hotkey to focused window |
| M4 | Candidate audio chains, swept against engines |
| M5 | `speechlab calibrate` — search, scoring, reports, config emission |

Two deliberate inversions of the spec's order:

- **The runtime ships before the optimizer.** A hotkey-to-dictation tool with a hand-picked
  model is useful on day one; a benchmark suite with nothing to configure is not. It also
  makes the project self-dogfooding, which is the only reliable source of the real-world
  audio the benchmark is meant to model.
- **Search comes last.** An optimizer needs a scoreable space, a known trial cost, and
  evidence that the score tracks real quality. None of those exist until the measurement
  spine does.

The highest-leverage unknown is spike 0.1: whether DSP chains can be applied *offline* to a
single recording with results equivalent to running the same chain live in PipeWire. If yes,
trial cost collapses from minutes to milliseconds and the whole search premise holds. If no,
the candidate space has to shrink drastically. See
[docs/roadmap.md](docs/roadmap.md) for the rest, including the open questions about the
composite score and the trial budget.

## Target platform

Primary: **Fedora + COSMIC + PipeWire + WirePlumber + Wayland.**
Secondary, once the primary works: GNOME, KDE Plasma, Sway, Hyprland.

Wayland global hotkeys and synthetic input are the known risk in the runtime, not the audio.

## Stack

Python — Typer, Pydantic, Rich, SQLite, ONNX Runtime, Matplotlib. Rust is deferred until a
profile indicts something specific.

## Contributing

Issues live on GitHub; labelling and triage conventions are in
[docs/agents/](docs/agents/). The four M0 spikes are open issues and all need Fedora
hardware and a human voice — if you have both, answering one of them is the most useful
thing anyone can do right now.
