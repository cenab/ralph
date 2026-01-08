# SUGGESTED-IMPROVEMENTS.md

This file documents structural issues identified during editing that cannot be fixed by rewriting prose alone. These require changes to the research, experiments, or paper structure.

---

## Issue: Missing Feature Ablation Study

**What is wrong:**
The paper claims that ML models "learn infrastructure-specific patterns (IP addresses, JA4 hashes) rather than AI-specific behaviors" (Section 6.4, line 617), but provides no systematic ablation study to verify this claim. The feature set includes ASN, JA4, organization, and shape correlation, but no experiment removes individual features to measure their contribution.

**Why this fails peer review:**
Reviewers will ask: "How do you know models rely on infrastructure features? Show me the ablation." Without ablation, the claim that poor generalization stems from infrastructure-specific learning is speculation, not demonstrated fact.

**Required structural change:**
Add an ablation study (Table or subsection) that:
1. Trains models with subsets of features (e.g., without JA4, without IP/org, with only shape features)
2. Reports internal accuracy and cross-version FPR for each feature subset
3. Demonstrates which features drive internal accuracy vs. which cause cross-version failure

**Affected sections:**
- Section 5 (Methodology) - add ablation design
- Section 6.7 (ML Results) - add ablation table
- Section 7.4 (Comparison of Detection Methods) - revise claim with ablation evidence

---

## Issue: Small Sample Size Limits Statistical Power

**What is wrong:**
The dataset contains 35-75 flows per scenario, only 3-6 TLS handshakes per scenario, and 38 message events per scenario. The ML evaluation uses only 3 runs with different random seeds. The resulting variance is extreme: FPR ranges from 2.7% to 78.7% (29x variation) across runs on the same pre-AI data.

**Why this fails peer review:**
Reviewers will note that confidence intervals are wide (acknowledged as "10.3% ± 11% at 95% confidence" in Section 8). The extreme variance across 3 runs suggests results are unstable and may not replicate. Reviewers may question whether findings reflect true patterns or sampling noise.

**Required structural change:**
Options (ranked by effort):
1. **Preferred**: Collect additional data to increase sample size per scenario to 100+ flows, 20+ TLS handshakes
2. **Alternative**: Report bootstrap confidence intervals for all key metrics (Jaccard, hit rates, FPR)
3. **Minimum**: Increase number of experimental runs from 3 to 10+ and report mean ± std for all cross-version metrics
4. Acknowledge explicitly in Limitations that sample size is a threat to external validity with quantified impact

**Affected sections:**
- Section 4 (Dataset) - note sample size limitations more prominently
- Section 6 (Results) - add confidence intervals to tables
- Section 8 (Limitations) - expand statistical power discussion

---

## Issue: Session-Level Splitting Not Fully Documented

**What is wrong:**
The paper mentions "session-level grouping: all flows from a single capture session remain together during splitting" (Section 5.6, line 345) but does not define what constitutes a "session" or document how many sessions exist per scenario. With only 35-75 flows per scenario, if sessions are small, the effective number of independent data points may be very low.

**Why this fails peer review:**
Reviewers concerned about data leakage will ask: "How many sessions are there? If there are only 2-3 sessions per scenario, then 70/30 splits mean 1-2 sessions in test, which is insufficient for reliable evaluation."

**Required structural change:**
1. Define "session" explicitly (single capture run? timestamp-bounded period?)
2. Report number of sessions per scenario in Table 2 (scenarios)
3. Confirm that train/test splitting is at session level, not flow level
4. If session count is low, acknowledge this limitation explicitly

**Affected sections:**
- Section 4.1 (Scenario Definitions) - add session count
- Section 5.6 (Evaluation Protocol) - clarify session definition
- Section 8 (Limitations) - if warranted

---

## Issue: Pre-AI Baseline Temporal Gap Creates Confounding

**What is wrong:**
The pre-AI baseline (Erdenebaatar et al., 2022) predates the AI-era captures (December 2024) by ~2 years. The paper acknowledges this creates potential confounding: "We treat pre-AI comparisons as indicative of infrastructure drift rather than controlled experimental contrast" (Section 4.5, line 275). However, the paper still uses pre-AI data as a primary validation set for cross-version claims.

**Why this fails peer review:**
Reviewers will note that the 2-year gap conflates multiple changes: AI feature integration, WhatsApp version updates, Android version differences, backend infrastructure evolution, and network conditions. The claim that detection fails due to "infrastructure drift" could equally be explained by temporal drift unrelated to AI.

**Required structural change:**
Options:
1. **Preferred**: Collect a new non-AI baseline using the same WhatsApp version as AI-era captures (December 2024) but with AI features disabled or unused. This would isolate AI effects from version effects.
2. **Alternative**: Frame pre-AI comparisons more cautiously throughout. Replace claims like "cross-version fragility" with "cross-temporal fragility" and acknowledge that AI vs. infrastructure effects cannot be disentangled.
3. **Minimum**: Add explicit limitation noting that pre-AI vs AI-era differences may reflect version/temporal drift rather than AI-specific effects

**Affected sections:**
- Section 4.1 (Scenario Definitions) - clarify what pre-AI baseline can/cannot show
- Section 6 (Results) - qualify cross-version claims
- Section 7 (Discussion) - adjust RQ4 answer to acknowledge confounding
- Section 8 (Limitations) - expand temporal confounding discussion

---

## Issue: No Comparison to Prior AI Detection Methods

**What is wrong:**
The paper cites Lyu et al. (2024) for ChatGPT detection via SNI filtering, but does not implement or compare against this approach. The related work discusses McDonald et al. (2025) "Whisper Leak" attack but does not evaluate whether similar side-channel techniques would work for WhatsApp AI detection.

**Why this fails peer review:**
Reviewers may ask: "Prior work detected ChatGPT via domain filtering. Did you try that? If Meta AI uses distinct domains (even within WhatsApp), domain filtering might work. Why wasn't this baseline implemented?"

**Required structural change:**
Options:
1. Implement SNI-based detection (checking for AI-specific subdomains) and report results
2. Explain why SNI-based detection is inapplicable (e.g., all WhatsApp traffic uses the same SNI regardless of AI features) with evidence
3. If the answer is "SNI does not discriminate" (as Table 3 suggests with shared SNI values), state this explicitly as a negative result

**Affected sections:**
- Section 2.3 (Detection of Generative AI) - note that domain filtering is inapplicable and why
- Section 6.2 (TLS Fingerprint Analysis) - explicitly test SNI-based detection
- Section 7 (Discussion) - position against prior methods

---

## Issue: Burst Threshold Lacks Validation

**What is wrong:**
The paper recommends "alerting on 1-second burst magnitudes exceeding 2,000 bytes" (Section 7.5, line 709) as a practical detection heuristic. However, this threshold is derived from the same data used to evaluate it. There is no held-out validation set for this threshold.

**Why this fails peer review:**
Reviewers will note that threshold selection on the test data constitutes data snooping. The threshold may be overfit to this particular dataset and may not generalize to other networks, time periods, or WhatsApp versions.

**Required structural change:**
Options:
1. **Preferred**: Collect an independent validation dataset to test the 2,000-byte threshold
2. **Alternative**: Derive threshold from a subset of data (e.g., Runs A and B) and validate on held-out data (Run C)
3. **Minimum**: Explicitly acknowledge that the threshold is data-derived and requires operational validation; remove the specific 2,000-byte recommendation from Table 9

**Affected sections:**
- Section 6.6 (Burst Signatures) - clarify threshold is descriptive, not validated
- Section 7.5 (Operational Recommendations) - soften threshold recommendation
- Table 9 - qualify or remove specific threshold value

---

## Summary

| Issue | Severity | Effort to Fix |
|-------|----------|---------------|
| Missing Feature Ablation | High | Medium (add experiments) |
| Small Sample Size | High | High (data collection) or Low (statistical methods) |
| Session Splitting Documentation | Medium | Low (documentation) |
| Pre-AI Temporal Confounding | High | High (new data) or Low (reframing) |
| No Prior Method Comparison | Medium | Low (add negative result) |
| Burst Threshold Validation | Medium | Medium (split data) or Low (reframing) |

The author should prioritize the ablation study and address the temporal confounding issue, as these are most likely to draw reviewer objections. Sample size limitations should be acknowledged with quantified confidence intervals.
