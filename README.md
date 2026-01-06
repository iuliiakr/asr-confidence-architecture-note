# Multilingual ASR confidence is not a safe decision signal

## Overview

This repository documents an integration-level risk in multilingual speech pipelines:
<b>ASR confidence scores are not numerically comparable across languages and models when used as control signals.</b>

In production systems, confidence is commonly used to gate automation, trigger human review, or drive corrective feedback. In multilingual settings, this assumption leads to non-deterministic behavior.


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


## Observed Divergence

A small evaluation across English, Hindi, and Ukrainian, using Google STT, Whisper, and Faster-Whisper, demonstrates that:
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

This pilot is intentionally small and illustrative. It is not intended to measure model accuracy or compare providers.
Its sole purpose is to demonstrate an integration-level behavior observed when ASR confidence scores are used as control signals in multilingual pipelines.

### Setup

<b>Languages:</b> English (en), Hindi (hi), Ukrainian (uk)

<b>Utterances:</b> Short, semantically simple commands (3–6 words)

<b>Speaker:</b> single speaker; consistent recording conditions

<b>ASR Systems</b>:
- Google Speech-to-Text
- OpenAI Whisper
- Faster-Whisper

<b>Confidence Signals:</b>
- Google STT: provider-reported confidence score
- Whisper / Faster-Whisper: average log probability per segment (used as a confidence proxy)

<b>Labels:</b> human-judged transcription correctness (binary)

## Sample Outputs (Illustrative Only)



### Google STT

| Lang | File | Human Reference | Model Output (Raw) | Confidence | Correct? |
| :--- | :--- | :--- | :--- | :--- | :--- |
| **🇺🇸 English** | en-1.wav | I will arrive tomorrow morning. | I will arrive tomorrow morning. | 0.98 | ✅ Correct |
| | en-2.wav | Please send me the file. | Please send me the file. | 0.98 | ✅ Correct |
| | en-3.wav | Call me in the evening. | Call me in the evening. | 0.90 | ✅ Correct |
| **🇮🇳 Hindi** | hi-1.wav | मैं कल सुबह पहुँचूँगी। | मैं कल सुबह पहुंचेंगे। | <span style="color: #2ECC71;">**0.90**</span> | ❌ Agreement Error |
| | hi-2.wav | कृपया मुझे फ़ाइल भेज दीजिए। | कृपया मुझे 5 भेज दीजिए। | **<span style="color: green;">0.89</span>** | ❌ Hallucination |
| | hi-3.wav | मुझे शाम को फोन करो। | मुझे शाम को फोन करो! | 0.96 | ✅ Correct |
| **🇺🇦 Ukrainian** | uk-1.wav | Я приїду завтра вранці. | Я приїду завтра вранці. | 0.92 | ✅ Correct |
| | uk-2.wav | Будь ласка, надішли мені файл. | будь ласка Надішли мені файл | 0.71 | ⚠️ Formatting Mismatch |
| | uk-3.wav | Зателефонуй мені ввечері. | зателефонуй мені ввечері | 0.83 | ⚠️ Formatting Mismatch |



### OpenAI's Whisper

| Language | File | Human Reference | Model Output (Raw) | Confidence | Correct? |
| :--- | :--- | :--- | :--- | :--- | :--- |
| **English** | en-1.wav | I will arrive tomorrow morning. | 0.61 | ✓ Correct |
| | en-2.wav | Please send me the file. | 0.57 | ✓ Correct |
| | en-3.wav | Call me in the evening. | 0.49 | ✓ Correct |
| **Hindi** | hi-1.wav | मैं कल सुबह पहुँचूँगी। | मैं कल सुबह पहुंचेंगे। | 0.44 | <span style="color:red">✗ Wrong)</span> |
| | hi-2.wav | कृपया मुझे फ़ाइल भेज दीजिए। | 0.01 | <span style="color:red">✗ Wrong)</span> |
| | hi-3.wav | मुझे शाम को फोन करो। | 0.08 | <span style="color:red">✗ Wrong)</span> |
| **Ukrainian** | uk-1.wav | Я приїду завтра вранці | 0.69 | ✓ Correct |
| | uk-2.wav | Будь ласка, надішли мені файл | 0.60 | <span style="color:red">✗ Wrong</span> |
| | uk-3.wav | Зателефонуй мені ввечері | 0.62 | <span style="color:red">✗ Wrong</span> |


*Faster-Whisper**

| Language | File | Human Reference | Model Output (Raw) | Confidence | Correct? |
| :--- | :--- | :--- | :--- | :--- | :--- |
| **English** | en-1.wav | I will arrive tomorrow morning. | 0.66 | ✓ Correct |
| | en-2.wav | Please send me the file. | 0.69 | ✓ Correct |
| | en-3.wav | Call me in the evening. | 0.55 | ✓ Correct |
| **Hindi** | hi-1.wav | मैं कल सुबह पहुँचूँगी। | मैं कल सुबह पहुंचेंगे। | 0.46 | <span style="color:red">✗ Wrong</span> |
| | hi-2.wav | कृपया मुझे फ़ाइल भेज दीजिए। | 0.34 | <span style="color:red">✗ Wrong</span> |
| | hi-3.wav | मुझे शाम को फोन करो। | 0.38 | <span style="color:red">✗ Wrong</span> |
| **Ukrainian** | uk-1.wav | Я приїду завтра вранці | 0.69 | ✓ Correct |
| | uk-2.wav | Будь ласка, надішли мені файл | 0.57 | <span style="color:red">✗ Wrong</span> |
| | uk-3.wav | Зателефонуй мені ввечері | 0.62 | <span style="color:red">✗ Wrong</span> |
