---
title: "HCI Lab 3 — Building and Debugging a Local LLM Talking Character"
date: 2026-11-08
draft: false
tags: [HCI, Unity, LLM, latency]
slug: "hci-lab-3"
categories: ["Human-Computer Interaction"]
toc: true
description: "A local Unity LLM character, interface debugging, generation limits, and a ten-trial single-participant study of latency and thinking visibility."
---

## Introduction

For this lab I built a local conversational character in Unity and investigated two HCI exercises: **Exercise 1 — Measure Latency** and **Exercise 4 — Show the Thinking?** The central challenge was not merely producing a response. The interface also had to communicate its state, remain usable during streaming, and distinguish a completed answer from a prematurely stopped generation. This report therefore focuses on implementation decisions, problems encountered, repairs, and the evidence used to check them. The final ten-trial study is reported as a small, exploratory case study rather than a generalizable user experiment.

## Environment and System Design

The application uses Windows 11, Unity 6000.0.85f2, URP, LLMUnity 3.0.3, and TextMeshPro. The computer has an Intel Core i7-14650HX, 16 GB RAM, and an RTX 5060 Laptop GPU; **inference was performed on CPU (GPU layers = 0)**. The local model is **DeepSeek-R1-Distill-Qwen-1.5B Q4_K_M GGUF**, from the source linked below, with a previously checked SHA-256 of `c3db87e9a0985c61896a03734ecce62b2eff21358da66753f840b0d287b57811`. No paid inference API was used.

The LLM service manages model execution; an agent manages prompts and conversational state; and `NPCController` connects the input field, Send/Enter, conversation display, scroll view, cancellation, and session reset. A diagnostic agent retains backend completion metadata. Each turn is recorded with timing, settings, raw output, and outcome. The formal configuration used a **2048-token generation budget**, **8192-token context**, **temperature 0.6**, and **seed 0**. Both ON and OFF generate model reasoning text: the toggle changes **visibility**, not the underlying model task.

Sources: [LLMUnity](https://github.com/undreamai/LLMUnity), [GGUF model source](https://huggingface.co/lmstudio-community/DeepSeek-R1-Distill-Qwen-1.5B-GGUF), [original model card](https://huggingface.co/deepseek-ai/DeepSeek-R1-Distill-Qwen-1.5B).

## Development Challenges, Repairs, and Verification

### 1. Getting the Unity components to work together

**Problem.** The initial integration required resolving API-name differences, importing TextMeshPro Essentials, and correcting test assembly/component placement. These were setup and compilation issues, not participant-study outcomes.

**Resolution.** The component and API references were aligned with the installed Unity/LLMUnity versions, the required TMP assets were imported, and the test assembly arrangement was corrected. The scene could then compile and run with the local model. The final recorded EditMode suite passed **39/39** tests. That test result is evidence for the covered code paths, not a substitute for checking the rendered interface in Play Mode.

### 2. A thinking toggle whose state was unclear

**Problem.** The first UI used a cyan image both as the toggle background and its checkmark. Clicking produced no readily distinguishable visual change. The label itself was not a reliable click target, and the user could not confidently tell whether thinking was ON or OFF. The controller read `isOn`, but the state was poorly communicated.

**Resolution.** The background and checkmark were separated; the entire row became clickable; and the label explicitly displays **Show Thinking: ON** or **Show Thinking: OFF**. If changed while a reply is generating, the choice applies to the **next turn**, avoiding a mid-response change in what the timing data means. The interface also distinguishes *Model-generated reasoning text* from *Final answer*, or shows only the latter when thinking is hidden. Chat text, input, metrics, and CSV controls were enlarged for readability. Streaming follows the bottom unless the reader scrolls upward.

**Verification.** State/feedback tests were included in the EditMode suite, and actual Unity Play Mode screenshots showed distinct ON and OFF labels and corresponding content. This is a concrete usability correction: a control is not useful if its current state cannot be recognized.

![Unity Play Mode with Show Thinking ON](/myblog/images/hci-lab3/unity-thinking-on.png)

*Show Thinking: ON in Unity Play Mode after the interface repair. Model-generated reasoning text is visually separated from the final answer, and the mode and latency metrics remain visible.*

![Unity Play Mode with Show Thinking OFF](/myblog/images/hci-lab3/unity-thinking-off.png)

*Show Thinking: OFF in Unity Play Mode after the interface repair. Only the final answer is shown, with an explicit OFF label and a hidden-text status message. These development screenshots illustrate the interface; they are not screenshots of the ten formal study trials.*

### 3. Generation ended before the final answer

**Problem.** A longer prompt exposed a failure under the original **512-token** generation limit. The model could spend the entire budget in a `<think>` section without producing `</think>` or a final answer. The old controller could mistake normal request termination for a successfully completed answer. This also made the screen appear stuck or filled with unfinished thinking.

**Diagnosis.** In a controlled probe using the same question and seed, the 512-token run stopped at **512 tokens** with backend stop type `limit`, an unclosed thinking block, and no final answer. Increasing the budget to **2048** produced **1289 tokens**, backend stop type `eos`, a closed thinking block, and a final answer. The 512-token output was an exact prefix of the longer output. This supports the generation-budget explanation for that controlled failure; the original problematic UI turn lacks sufficient raw metadata to prove that it failed at exactly 512 tokens.

**Resolution.** The generation budget was raised to 2048 while keeping context at 8192 and temperature at 0.6. The controller now checks backend stop information and answer structure. An unclosed thinking section, missing final answer, token-limit stop, or context truncation is marked **incomplete**, not **success**. Incomplete responses are excluded from subsequent conversation history. Per-turn raw output, settings, generated-token count, stop reason, and status are saved for inspection.

**Verification.** A controlled long-prompt comparison and subsequent actual ON/OFF model-controller runs produced complete answers. The original failed turn and older 512-token data were retained, but excluded from the formal 2048-token analysis. Raising the limit reduces this observed failure; it does not guarantee every future response will be complete.

### 4. Streaming tags and what “first token” means

**Problem.** Stream callbacks do not necessarily align with the boundaries of `<think>` and `</think>`. A tag may arrive in fragments. In addition, the first backend text update is not necessarily the first text visible to a user, especially when thinking is hidden.

**Resolution.** A state-machine parser handles cumulative streaming snapshots and fragmented tags, preventing partial control tags from being shown as ordinary answer text. The application records three distinct timing quantities with a monotonic `Stopwatch`: **Raw TTFT**, the first observed nonempty changed streaming callback; **Visible TTFT**, the first non-whitespace model content in the selected display; and **Total Response Time**, the duration of the request. These are application-level measurements. Raw TTFT is not a hardware measurement of the exact instant a token was computed, and Visible TTFT is not an eye-tracking or monitor-refresh measurement. Model loading before a ready service is not included in a turn.

**Verification.** Parser, streaming, and incomplete-response cases were covered by tests, while ON/OFF runtime trials confirmed that the hidden-thinking mode can have a long delay before the first visible answer despite early backend output.

### 5. Keeping measurements and subjective ratings auditable

**Problem.** Earlier development probes, faulty runs, and formal responses all produced files. Mixing them would bias the results. The manually recorded ratings also did not initially contain Session/Turn IDs.

**Resolution.** The ten ratings were preserved as entered. Each was matched using the **exact question text and ON/OFF mode**, then checked against backend prompt content and CSV/diagnostic **Session ID and Turn ID**. All ten available formal matches were unique; timestamps supported the reported round order but were not used to guess matches by row position. Records from the old generation budget, debugging, and incomplete runs were retained but excluded. A repeatable analysis script and source-file hashes support later checking.

**Verification and caveat.** All ten matched formal turns used `n_predict=2048`, stopped with `eos`, had closed thinking blocks and nonempty final answers, and were marked successful without reported context truncation. Because the rating sheet lacked direct IDs and timestamps, this matching cannot rule out unlogged repetitions; future rating forms should capture identifiers at the time of evaluation.

## Experimental Procedure


One participant completed ten ratings, with each of five questions asked once in each display mode. Waiting uses a five-point scale from short to long and trust a five-point scale from low to high. The source rating table contains no participant-level replication. ON was presented first for three pairs and OFF first for two. The available evidence does not establish randomized order.

- **Q1**: What is human-computer interaction?
- **Q2**: Why is usability important?
- **Q3**: What is augmented reality?
- **Q4**: How can AI support education?
- **Q5**: What makes an interface trustworthy?

| Question | Order | Waiting ON / OFF | Trust ON / OFF |
| --- | --- | --- | --- |
| Q1 | ON then OFF | 1 / 5 | 4 / 3 |
| Q2 | OFF then ON | 1 / 4 | 3 / 3 |
| Q3 | ON then OFF | 1 / 3 | 3 / 3 |
| Q4 | OFF then ON | 1 / 4 | 4 / 4 |
| Q5 | ON then OFF | 1 / 3 | 4 / 3 |

Each rating was matched by its exact question text and ON/OFF mode to the last user message in a backend prompt. CSV and diagnostic files were then joined by Session ID and Turn ID. There was one candidate per question-mode key, producing 10/10 valid matches. Timestamps independently agreed with the reported round order, but were not used to assign records by position. The ratings themselves lack timestamps and session identifiers, so this mapping depends on uniqueness among the available logs and cannot rule out unlogged repetitions.

All matched formal trials have n_predict=2048 in both the saved parameters and backend metadata, success status, EOS stopping, nonempty final answers, closed thinking blocks and no backend context truncation. Each prompt contains one user message, supporting fresh conversation history. The first ON trial is labelled cold-first-request; the other nine are subsequent requests. This imbalance is retained and examined separately.

## Results


The table summarizes five trials per condition. SD is the sample standard deviation across repeated trials from one person. Means of ordinal ratings are descriptive conveniences and are accompanied by medians and pair-level scores.

| Measure | ON mean (SD) | OFF mean (SD) | ON median | OFF median |
| --- | --- | --- | --- | --- |
| Raw TTFT (ms) | 125.79 (109.50) | 76.04 (16.20) | 89.35 | 68.38 |
| Visible TTFT (ms) | 224.64 (108.24) | 21511.35 (9186.76) | 187.34 | 24023.99 |
| Total response time (ms) | 28211.93 (20151.33) | 26453.93 (11518.04) | 29418.17 | 27129.12 |
| Perceived waiting (1–5) | 1.00 (0.00) | 3.80 (0.84) | 1.00 | 4.00 |
| Trust (1–5) | 3.60 (0.55) | 3.20 (0.45) | 4.00 | 3.00 |

| Round | Question | Mode | Session prefix | Raw TTFT ms | Visible TTFT ms | Total ms | Generated tokens | Stop |
| --- | --- | --- | --- | --- | --- | --- | --- | --- |
| 1 | Q1 | ON | de23e4af | 320.228 | 416.615 | 56726.475 | 1212 | eos |
| 2 | Q1 | OFF | 53ea118c | 68.376 | 30740.940 | 36431.615 | 753 | eos |
| 3 | Q2 | OFF | e2caf64e | 94.742 | 24023.992 | 27129.123 | 563 | eos |
| 4 | Q2 | ON | ed37257b | 63.980 | 161.134 | 10828.721 | 226 | eos |
| 5 | Q3 | ON | 643937cd | 91.485 | 193.647 | 7354.957 | 151 | eos |
| 6 | Q3 | OFF | 2b2652e6 | 59.801 | 8181.023 | 11746.502 | 242 | eos |
| 7 | Q4 | OFF | 4edc1664 | 92.137 | 28098.899 | 38633.117 | 798 | eos |
| 8 | Q4 | ON | 8e58c29c | 63.931 | 164.452 | 36731.333 | 792 | eos |
| 9 | Q5 | ON | be5f0159 | 89.349 | 187.340 | 29418.172 | 609 | eos |
| 10 | Q5 | OFF | 0812ab8f | 65.161 | 16511.903 | 18329.271 | 379 | eos |

![Paired objective latency](/myblog/images/hci-lab3/paired-latency.png)

![Waiting and trust ratings by question](/myblog/images/hci-lab3/subjective-ratings.png)

The mean waiting rating was 1.0 for ON and 3.8 for OFF. Every ON trial received 1, while OFF ranged from 3 to 5. ON had lower waiting scores in all five question pairs. Trust averaged 3.6 for ON and 3.2 for OFF. Trust was higher under ON in two pairs and tied in three. No participant-population significance claim is made.

## HCI Interpretation

The clearest finding is about **feedback during waiting**, not raw model speed. ON makes generated reasoning text visible early, whereas OFF withholds that text until a final answer is available. In the ten observations, ON had a much shorter mean Visible TTFT (**0.225 s** versus **21.511 s**) and lower perceived waiting scores (**1.0** versus **3.8**), even though its mean total response time was not shorter (**28.212 s** versus **26.454 s**). The descriptive pooled Spearman correlation between Visible TTFT and waiting was **0.931**, versus **0.169** for total duration. These are dependent observations and the display mode itself is a major confounder; the correlation is not causal evidence.

Trust was less consistent. Mean trust was **3.6** in ON and **3.2** in OFF, with an increase in only **two** matched pairs and ties in **three**. The study therefore does not show that displaying thinking reliably improves trust. Furthermore, model-generated reasoning text is **not a verified transcript of the model's internal computation**. The UI should describe it accurately rather than presenting it as privileged access to the model's true reasoning.

The first ON trial was marked cold-first-request. Excluding both Q1 turns, mean Raw TTFT was approximately **77.19 ms (ON)** versus **77.96 ms (OFF)**. This sensitivity check helps illustrate why a first-run cost should not be mistaken for a display-mode effect.

## Limitations and Next Steps

This was **one participant**, not ten independent users. The five paired questions provide repeated observations from the same person. Order was **ON first for three pairs and OFF first for two**, without verified randomization; repetition, expectation, learning, and fatigue could affect ratings. Even with the same settings, the paired model replies differed in content and length. The cold-first-request flag occurred only in ON. Five-point scales are ordinal, ON waiting scores were all at the minimum (a floor effect), and no independent correctness assessment or qualitative comments were collected. The rating sheet also lacked direct trial identifiers. Consequently, neither the observed waiting difference nor the modest trust difference should be generalized to other users or interpreted as a causal effect.

A stronger follow-up would recruit more participants, counterbalance and randomize presentation order, record trial IDs with ratings, compare answer correctness and length, and include qualitative feedback. It should also distinguish perceived waiting from objective response completion time.

## Conclusion

The main engineering lessons were that **UI state must be legible**, **request termination must not be confused with a complete answer**, and **backend latency must be distinguished from visible feedback**. The resulting application provides a local, inspectable basis for the two HCI exercises. In this limited case study, showing model-generated reasoning was associated with earlier visible output and lower reported waiting, but not with a consistent improvement in trust. The development failures and their repairs are part of the evidence, not details to erase from the report.
