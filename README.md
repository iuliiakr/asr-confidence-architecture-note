# Multilingual ASR confidence is not a safe decision signal

## Overview

This repository documents an integration-level risk in multilingual speech pipelines:
<b>ASR confidence scores are not numerically comparable across languages and models when used as control signals.</b>

In production systems, confidence is commonly used to gate automation, trigger human review, or drive corrective feedback. In multilingual settings, this assumption leads to non-deterministic behavior.

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

## Observed Divergence

A small evaluation across English, Hindi, and Ukrainian—using Google STT, Whisper, and Faster-Whisper—demonstrates that:
- Confidence distributions differ significantly by language for equivalent utterances
- Incorrect transcriptions may receive higher confidence than correct ones in other languages
- Model-specific confidence proxies are not aligned

As a result, identical numeric thresholds encode different reliability levels depending on language and model.

&nbsp;

## System-Level Impact

Using raw ASR confidence as a control signal introduces:
- inconsistent automation gating across languages,
- false corrective feedback in tutoring systems,
- unstable human review rates in data pipelines,
- unpredictable cost and UX behavior in global deployments.

The failure mode is architectural, not model-specific.

&nbsp;

## Architectural Implication

Multilingual speech systems require an explicit <b>Confidence Normalization Layer</b> that:
- calibrates confidence per language and model,
- maps raw scores to task-specific reliability bands,
- and prevents provider-specific semantics from leaking across system boundaries.

Confidence should be treated as an interface contract, not a raw metric.

&nbsp;

## Appendix: Sample Transcriptions and Confidence Outputs

### Setup

Languages: English (en), Hindi (hi), Ukrainian (uk)
Speaker: single speaker (constant recording conditions)

Models evaluated:
- Google Speech-to-Text
- OpenAI Whisper

Labels: human-judged transcription correctness (binary)

Signals analyzed: model-reported or derived confidence values




