# Multilingual ASR confidence is not a safe decision signal

## Motivation

In multilingual voice architectures, confidence is rarely a standardized or comparable signal across language models. While designing a pronunciation tutoring feedback loop, I observed that applying uniform confidence thresholds across languages led to inconsistent system behavior—triggering false corrections in some languages while missing clear errors in others. This surfaced a broader integration challenge: how to normalize heterogeneous ML confidence signals to deliver a consistent user experience without introducing brittle, per-language logic.


