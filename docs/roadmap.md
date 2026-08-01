# VoiceCal roadmap

Decomposition of [docs/idea.md](idea.md) into buildable slices.

## Ordering principle

The spec is written as seven modules. Building module-by-module produces six
finished components and nothing that runs. Every milestone below instead ends
in a command a user can execute, and each one subsumes parts of several spec
modules.

Two consequences worth stating up front:

- **Runtime mode ships before the optimizer.** A hotkey-to-dictation tool with a
  hand-picked model is useful on day one; a perfect benchmark suite with no
  runtime is not. It also makes the project self-dogfooding, which is the only
  reliable source of the real-world audio the benchmark is supposed to model.
- **Search comes last.** An optimizer needs a scoreable space, a trial cost, and
  evidence that the score correlates with real quality. None of those exist
  until M1–M4 do.

## M0 — Spikes

Throwaway code answering questions whose answers change the architecture. Each
spike is done when it has produced a written answer, not a component.

| # | Question | What it changes |
|---|---|---|
| 0.1 | Can candidate DSP chains be applied **offline** to one recording with results equivalent to processing the same chain live in PipeWire? | If yes: record once, evaluate hundreds of chains as pure file transforms. Trial cost collapses from minutes to milliseconds and M4 becomes tractable. If no: every trial needs a live graph rebuild and re-read, and the candidate space must shrink drastically. **Highest-leverage question in the project.** |
| 0.2 | Can a PipeWire filter chain be built, used, and torn down programmatically, repeatably, without leaking nodes? | Feasibility of the whole "test N routings" premise, and of config emission in M5. |
| 0.3 | Does WER on a read passage predict transcription quality on spontaneous speech? | If correlation is weak, the ground-truth corpus needs spontaneous samples and the objective function needs rethinking. |
| 0.4 | Is per-trial CPU/RAM attribution reliable enough to score on? | Whether the composite score keeps its resource terms or drops to accuracy + latency only. |

## M1 — Measurement spine

`voicecal record` and `voicecal bench` against one fixed pipeline and one fixed
engine, writing a result row and printing WER and latency.

Everything later plugs into this. Contents:

- Ground-truth corpus and the passage-reading flow (spec Module 1).
- Recording via PipeWire with fixed parameters (Module 2).
- **Transcript normalisation.** Subtler than it looks and it silently sets the
  accuracy floor: `python -m venv .venv` must not be scored as a failure
  against `python dash m venv dot venv`. Spoken-form mapping for punctuation,
  symbols, numbers, and command-line syntax is its own unit of work.
- WER/CER scoring.
- SQLite result schema — trial, config, metrics, timestamps.
- CLI skeleton: Typer, Pydantic config models, Rich output.

## M2 — Engine breadth

Pluggable STT interface plus faster-whisper, whisper.cpp, and one non-Whisper
engine (Sherpa-ONNX or Vosk) to prove the interface is not Whisper-shaped.
Sweep engine × model size over a fixed recording.

First genuinely useful output: *which model should I run on this machine*.

## M3 — Runtime mode

Hotkey → record → transcribe → paste into the focused application, on Wayland,
with an optional auto-submit. Consumes a config file that M2 can already
produce by hand.

Wayland global hotkeys and synthetic input are the risk here, not the audio.
Scope to COSMIC first, per the spec's primary target.

## M4 — Chain breadth

Candidate audio-chain generation and application, in whichever mode spike 0.1
selected. Sweep chain × engine. Report which processing steps actually earn
their place — the spec asserts several will; this is where that gets tested.

## M5 — Search and reporting

- Random search and grid over the joint space, with a trial budget.
- Composite scoring (see open question below).
- Report generation and graphs.
- Config emission: `pipeline.toml`, `voice.conf`, `models.toml`, PipeWire and
  WirePlumber fragments.

`voicecal calibrate` — the headline command — exists at the end of this
milestone and not before.

Bayesian optimisation and TPE are only worth their complexity if the per-trial
cost after spike 0.1 stays high. Decide then, on the measured cost.

## Deferred

Not in v1, and each needs the milestones above to exist first:

- LLM analyst over benchmark results (spec Module 6).
- End-to-end assistant benchmark: TTFT, time to first spoken response (Module 7).
- Remote phone microphone, wake word, VAD, continuous dictation, command and
  macro modes.
- Desktop breadth beyond COSMIC.
- **Rust.** The spec names it for "performance-critical modules" before any
  measurement exists. Ship Python; port only what a profile indicts.
- **The plugin architecture as a stated goal.** M2 and M4 each force exactly one
  real extension point. Extension points invented ahead of a second implementer
  are guesses.

## Open questions

**The composite score in the spec is wrong as written.** It sums
`0.50 × accuracy + 0.30 × latency + 0.10 × cpu + 0.10 × memory` with positive
weights while the stated intent is to minimise the last three. Optimising that
expression maximises latency. The terms need normalising to a common
"higher-is-better" scale, or the cost terms need negative weights and units
that make them commensurable with a percentage.

**The trial budget contradicts the ten-minute promise.** Engines × model sizes ×
audio chains × sample rates is thousands of combinations. Ten minutes of
wall-clock is only reachable if spike 0.1 succeeds *and* search is bounded.
Otherwise the headline needs to change or calibration becomes an overnight job.

**Success metrics are unmeasurable as stated.** "Reduce transcription error
compared to default settings" needs a defined baseline: distro-default
PipeWire, no processing, and a named default model. Pin it in M1 so every later
number has something to be better than.
