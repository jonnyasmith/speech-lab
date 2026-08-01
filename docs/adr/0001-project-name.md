# The project is named SpeechLab

The specification was drafted under the name VoiceCal; the repository was created as
`speech-lab`. We are keeping SpeechLab as the single name for the project, the CLI, and the
eventual package, and retiring VoiceCal entirely.

## Considered Options

**VoiceCal** is the more specific name and reads like a product, which normally wins. Two
things disqualify it.

It is one character from "voicecall", a term of art in telephony with established Linux
projects already using it (`nemomobile/voicecall`, `sailfishos/voicecall`) — a permanent
search and typo tax in precisely the Linux-desktop audio space this project targets.

More importantly, "Cal" names the calibration phase, which is the part of the workflow a
user stops using. [The roadmap](../roadmap.md) deliberately ships the dictation runtime at
M3, before the optimizer, because that is the half people touch daily; calibration is run
once. Naming the whole project after its setup step mis-scopes it, and would leave the
runtime permanently sounding like a bolt-on to a benchmark tool.

**SpeechLab** is vaguer, and collides with how academic speech groups name themselves
(NTU Speechlab, `lucasondel/SpeechLab`). That cost is real but smaller: those are research
groups and script collections, not tools competing for the same install command.

Its vagueness is also load-bearing here. The project is a benchmark harness *and* a
dictation runtime, with the spec's stretch goal reaching toward a broader AI-interaction
benchmark suite. A container name covers all of that; VoiceCal covers one milestone of it.

**Inventing a third name was ruled out on evidence.** `earshot`, `utter`, `diction`, and
`attend` are all taken on PyPI. Both existing candidates are free there, so there is no
availability argument for starting over.

## Consequences

The CLI is `speechlab`, with `speechlab calibrate` and the runtime as sibling subcommands,
so the benchmark and dictation halves need no second name between them.
