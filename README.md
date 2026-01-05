# Multilingual ASR confidence is not a safe decision signal

## Overview

This artifact highlights a critical integration risk in multilingual speech pipelines: ASR confidence scores are not comparable across languages and models when used as control signals.

In many production architectures, ASR confidence is treated as a standardized metric to gate automation, trigger human review, or control feedback loops. This assumption breaks down in multilingual systems, where identical numeric thresholds produce materially different behavior depending on language, script, and model provider.

This repository presents a small, reproducible evaluation demonstrating how uncalibrated confidence scores destabilize downstream workflows and motivates the need for a Confidence Normalization Layer in global voice architectures.

&nbsp;

## Architectural Context

This issue emerges in multiple common architectures, including:

1. Pronunciation Feedback Loops (Interactive Systems)
- ASR confidence determines whether learner pronunciation is corrected
- Miscalibrated confidence leads to false corrections or missed errors
- Effects are immediate and user-visible

2. TTS Corpus Creation Pipelines (Data Systems)
- ASR confidence gates automatic acceptance vs. human review
- Language-dependent confidence skew inflates validation costs
- Bias introduced at ingestion propagates into training data

Although the downstream systems differ, the architectural risk is the same: confidence is used as a decision signal without normalization across languages.

&nbsp;

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


