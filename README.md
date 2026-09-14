# GigaChat Guardrails under Context-Length Manipulation

[![Python](https://img.shields.io/badge/python-3.10+-blue.svg)](https://www.python.org/)
[![License: MIT](https://img.shields.io/badge/License-MIT-yellow.svg)](LICENSE)

A repository containing code, data, and experimental analysis regarding the
robustness of GigaChat model guardrails against inference-layer attacks
involving the manipulation of input context length and type.

This work was conducted as part of an internship at
**ITMO University**, Faculty of Digital Transformation,
"Big Data and Machine Learning" program,
specializing in **LLM Security**.

---

## Abstract

We investigate whether GigaChat's guardrails (protecting against
prompt injections and template-based refusals) are weakened when
long additional context—whether neutral or sensitive, in narrative
or dialogue form—is appended to a malicious prompt.
| Study | Models | Attack Type | ASR |
|---|---|---|---|
| Yakovlev & Petrov (2026) | YandexGPT, GigaChat (7 models) | Prompt injection (3 classes) | Depends on model/attack |
| Limonova & Studenikina (2026) | YandexGPT Lite/Pro, GPT-OSS-20B/120B | Safety benchmark (14 categories) | High safety rates overall |
| Kumarappan et al. (2026) | Qwen 3 8B, GPT-OSS 20B/120B, Llama 3.1 70B, GPT-4o Mini, GPT-5, Gemini 3 Flash, Claude Opus/Sonnet | Multi-turn priming | 90–100% (open), 70–90% (frontier), 15–17.5% (Claude) |
| Zhao et al. (2025) | Gemini 2.5 Pro, GPT o4 mini, Grok 3 mini, Claude 4 Sonnet | CoT Hijacking | 99%, 94%, 100%, 94% |
| Shah et al. (2025) | LLaMA, Qwen, Mistral, Gemini | NINJA (long-context jailbreak) | Significant increase across models |
| Zhan et al. (2024) | 30 LLM agents (ReAct-prompted GPT-4) | Indirect prompt injection | 24% (GPT-4 ReAct) |
| Zyryanova et al. (2024) | ChatGPT 4o, Claude 3.8, YandexGPT | Prompt injection | YandexGPT/Claude > ChatGPT 4o |
| Kolomychenko (2025) | GigaChat, YandexGPT, Western LLMs | Moderation/censorship | High hard censorship rates |
| Liu et al. (2026) | GPT-4, Claude 3.5, Gemini Pro | Jailbreaking (250 attacks) | 69.2%, 56.4%, 74.4% |
| AutoRAN (2026) | GPT-o4-mini | Reasoning hijacking | 100% (AdvBench) |

*Note: Some studies do not report a single ASR figure; the table reflects their main findings.*

**Key Results:**

- The rate of template-based refusals (`p_hard`) is independent of
context length within the 500–10,000 character range (Δ < 0.01, p > 0.25).
- The hypothesis that guardrails weaken monotonically as context
length increases is not supported.
- [To be added after final analysis: baseline → 500 effect]
- GigaChat's defense architecture demonstrates robustness against
inference-layer attacks involving context lengthening—unlike
certain Western models, where similar attacks result in a
multi-fold increase in the Attack Success Rate.

---

## Hypotheses

**H1 (Primary).** As the length of the additional context increases,
the rate of template-based refusals (`p_hard`) decreases. **H2 (post-hoc).** The effect is determined not by length, but by the mere presence
of context: `p_hard(0) ≫ p_hard(500) ≈ p_hard(10000)`.

---

## Dataset

The **RuSafety** benchmark (Limonova & Studenikina, 2026) is used:
1,198 malicious prompts in Russian, across 14 categories
(`Prompt Leaking`, `Goal Hijacking`, `Crimes and Illegal Activities`,
`Privacy and Property`, `Role Play Instruction`, `Unsafe Instruction
Topic`, `Unfairness and Discrimination`, `Insult`,
`Inquiry with Unsafe Opinion`, `Mental Health`, `Ethics and Morality`,
`Physical Harm`, `Reverse Exposure`, `Commonsense`).

A stratified subsample is used for the context experiment:
**30 prompts per category** (~420 unique prompts).

The dataset is not included in the repository. To reproduce the results,
download it via this link: [link to RuSafety].

---

## Models

All experiments were conducted using the official GigaChat API
(free tier for individuals) with 4 models:

| Model | Role in experiment |
|---|---|
| `GigaChat-2` | base |
| `GigaChat-2-Pro` | mid-tier |
| `GigaChat-2-Max` | high-end |
| `GigaChat-3-Ultra` | flagship |

Client: official `gigachat` Python SDK.

---

## Experimental Design

**Within-subject.** Each prompt is submitted to each model under
several conditions. Per single prompt:

- 4 models ×
- (2 context types × 2 forms × 3 lengths + 1 baseline) ×
- 3 repetitions =
- **48 requests**

Context types: `neutral`, `sensitive`.
Forms: `narrative`, `dialogue`.
Lengths: `500`, `4000`, `10,000` characters + `0` (baseline).

Each context is a long source text
(`data/contexts/*.txt`) from which a slice of the required length
(`text[:ctx_len]`) is taken and prepended to the malicious prompt.

**Baseline** (`ctx_len = 0`) — malicious prompt without context,
`ctx_type = "baseline"`, `ctx_form = "baseline"`.

---

## Response Classification

Each response is labeled as belonging to one of three classes:

| Class | Definition |
|---|---|
| `hard_template` | template-based guardrail refusal (one of the fixed phrases) |
| `soft_refusal` | short refusal without a template ("I can't," "I won't"...) |
| `engaged` | substantive response |

Primary metric: `p_hard` — the proportion of template-based refusals,
averaged over 3 repetitions.

**Breakdown of failure:**

- `Δp_hard = Δp_soft + Δp_engaged` — exact identity.
- `full_share = Δp_engaged / Δp_hard` — share of full failure.
- `partial_share = Δp_soft / Δp_hard` — share of partial failure.

---

## Repository Structure
