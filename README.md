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

A small evaluation across English, Hindi, and Ukrainian, using Google STT and Whisper, demonstrates that:
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

Raw confidence values encode provider- and language-specific semantics and cannot be consumed directly as decision signals without introducing non-deterministic system behavior.

Multilingual pipelines therefore require an explicit normalization boundary that translates model-specific confidence into task-level reliability guarantees.

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

<b>Confidence Signals:</b>
- Google STT: provider-reported confidence score
- Whisper: average log probability per segment (used as a confidence proxy)

<b>Labels:</b> human-judged transcription correctness (binary)

&nbsp;

## Sample Outputs (Illustrative Only)


### Google STT

In the samples below, several Hindi transcriptions are assigned high confidence despite semantic or grammatical errors, illustrating that confidence values are not directly comparable across languages.

| Language | File | Human Reference | Model Output (Raw) | Confidence | Human Judgment |
| :--- | :--- | :--- | :--- | :--- | :--- |
| **🇺🇸 English** | en-1.wav | I will arrive tomorrow morning. | I will arrive tomorrow morning. | 0.98 | ✅ Correct |
| | en-2.wav | Please send me the file. | Please send me the file. | 0.98 | ✅ Correct |
| | en-3.wav | Call me in the evening. | Call me in the evening. | 0.90 | ✅ Correct |
| **🇮🇳 Hindi** | hi-1.wav | मैं कल सुबह पहुँचूँगी। | मैं कल सुबह पहुंचेंगे। | 0.90 | ❌ Agreement Error |
| | hi-2.wav | कृपया मुझे फ़ाइल भेज दीजिए। | कृपया मुझे 5 भेज दीजिए। | 0.89 | ❌ Hallucination |
| | hi-3.wav | मुझे शाम को फोन करो। | मुझे शाम को फोन करो! | 0.96 | ✅ Correct |
| **🇺🇦 Ukrainian** | uk-1.wav | Я приїду завтра вранці. | Я приїду завтра вранці. | 0.92 | ✅ Correct | 
| | uk-2.wav | Будь ласка, надішли мені файл. | будь ласка Надішли мені файл | 0.71 | ⚠️ Formatting Mismatch |
| | uk-3.wav | Зателефонуй мені ввечері. | зателефонуй мені ввечері | 0.83 | ⚠️ Formatting Mismatch |

&nbsp;


### OpenAI's Whisper (large-v3)

Whisper does not expose an explicit confidence score. The avg_logprob shown below is Whisper’s average token log-probability for the decoded output and is commonly used as a proxy for internal model confidence. The examples illustrate that similar or higher log-probability values may correspond to both correct and incorrect transcriptions, depending on language and linguistic features.

| Language | File | Human Reference | Model Output (Raw) | avg_logprob | Human Judgment |
| :--- | :--- | :--- | :--- | :--- | :--- |
| **🇺🇸 English** | en-1.wav | I will arrive tomorrow morning. | I will arrive tomorrow morning. | -0.18 | ✅ Correct |
| | en-2.wav | Please send me the file. | Please send me the file. | -0.35 | ✅ Correct |
| | en-3.wav | Call me in the evening. | Call me in the evening. | -0.49 | ✅ Correct |
| **🇮🇳 Hindi** | hi-1.wav | मैं कल सुबह पहुँचूँगी। | में कल सुबे पखुचुंगी | -0.22 | ❌ 1 phonetic error |
| | hi-2.wav | कृपया मुझे फ़ाइल भेज दीजिए। | कृपया मुझे फाइल भेज दीचिये  | -0.15 | ❌ 1 phonetic error |
| | hi-3.wav | मुझे शाम को फोन करो। | मुझे शाम को फोन करो | -0.11 | ✅ Correct |
| **🇺🇦 Ukrainian** | uk-1.wav | Я приїду завтра вранці. | Я приїду завтра вранці. | -0.15 | ✅ Correct |
| | uk-2.wav | Будь ласка, надішли мені файл. | Будь ласка, на дешлий маніфайл. | -0.17 | ❌ Hallucination |
| | uk-3.wav | Зателефонуй мені ввечері. | Зателефонуй мені ввечері. | -0.11 | ✅ Correct |


Human Judgment - these correctness labels reflect comparison between the raw model output and the human reference transcript:
- ✅ Correct: Semantically and grammatically equivalent
- ⚠️ Formatting Mismatch: Punctuation, casing, or spacing differences without semantic impact
- ❌ Agreement Error: Grammatical error affecting correctness
- ❌ Hallucination: Lexical insertion, substitution, or meaning change
&nbsp;
