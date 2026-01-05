# Multilingual ASR confidence is not a safe decision signal

## Motivation

In multilingual voice architectures, confidence is rarely a standardized or comparable signal across language models. While designing a pronunciation tutoring feedback loop, I observed that applying uniform confidence thresholds across languages led to inconsistent system behavior - triggering false corrections in some languages while missing clear errors in others. This surfaced a broader integration challenge: how to normalize heterogeneous ML confidence signals to deliver a consistent user experience without introducing brittle, per-language logic.

## Scope of This Note

This is <b>not a model benchmark or accuracy comparison</b>.
The <b>focus is on system behavior and architectural risk</b> when confidence values are used for downstream decisions.

## Pilot Setup

Languages: English (en), Hindi (hi), Ukrainian (uk)
Speaker: single speaker (constant recording conditions)

Models evaluated:
- Google Speech-to-Text
- OpenAI Whisper

Labels: human-judged transcription correctness (binary)

Signals analyzed: model-reported or derived confidence values

## Key Observation

Across models, confidence scores are not consistently aligned with correctness across languages.

Observed failure patterns include:
- High confidence with incorrect transcription (Hindi, Ukrainian)
- Low confidence with correct transcription (English)
- Script confusion with strong internal certainty (Whisper)

As a result, identical confidence thresholds produce language-dependent behavior.

## Why This Is a System Risk

In production architectures, ASR confidence is commonly used to:
- auto-accept or reject transcriptions
- trigger user feedback or corrections
- route requests to human review

When confidence is treated as a calibrated, language-agnostic signal:
- errors propagate silently for some languages
- non-English users receive inconsistent UX
- system health metrics remain misleadingly “green”

## Architectural Takeaways

### Risky patterns
- Global confidence thresholds
- Language-blind routing or feedback logic

### Safer approaches
- Language-aware thresholds
- Explicit language and script enforcement
- Treating confidence as a weak heuristic, not a guarantee
- Combining confidence with additional validation signals

## Applicability Beyond Tutoring

This risk applies to any multilingual voice system, including:
- customer support automation
- voice assistants
- accessibility tooling
- language assessment platforms

## Status

This note documents a small, controlled pilot intended to surface architectural risk.
Future work could explore confidence calibration and downstream decision simulation, but the core insight is already evident.


