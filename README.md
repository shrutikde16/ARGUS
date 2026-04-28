# ARGUS
### Assessing Reasoning, Grounding, and Uncertainty in Systems

> **A metacognition benchmark for frontier LLMs** — operationalizing the Nelson & Narens (1990) monitor-and-control loop to evaluate whether models can detect, localize, and repair their own reasoning failures.

**Shrutik De** · University of Illinois Urbana-Champaign  
*B.S. Statistics & Computer Science / B.S. Cognitive Psychology*

Submitted to the [Google DeepMind × Kaggle AGI Evaluation Hackathon](https://www.kaggle.com/competitions/kaggle-measuring-agi)

---
## Environment
This was done in Kaggle's Benchmark interface. Instead of needing a private API key for each of the models, you can use their interface and get $10 of free credits daily to test models.

## Motivation

Current benchmarks measure whether a model gets the right answer — not whether it understands *why* it was wrong or can act on that understanding. AGI requires **metacognition**: the capacity to monitor your own reasoning, recognize when it's failing, and self-correct.

ARGUS addresses this gap by isolating both halves of the Nelson & Narens (1990) cognitive loop:

- **Monitoring** — detecting that something is wrong
- **Control** — acting on that detection

*Think of a student mid-exam. Noticing that their answer contradicts an earlier step is monitoring. Crossing it out and redoing it is control.*

---

## Benchmark Design

### Three-Turn Structured Dialogue

Each item presents a reasoning chain (either flawed or clean) and elicits a structured three-turn response:

| Turn | Name | What It Measures |
|------|------|-----------------|
| **Turn 1** | Prospective Monitoring | Pre-task familiarity and difficulty prediction — *knowing what it knows* |
| **Turn 2** | Spontaneous Monitoring | Free-form observation of the provided chain; LLM-judged for error detection quality |
| **Turn 3** | Control | Structured output: error judgment · confidence score · step localization · error category · corrected solution |

### Dataset

**200 items** scaffolded from MATH and SQuAD using Gemini, balanced 1:1 between flawed and flawless chains.

| Property | Detail |
|----------|--------|
| Domains | 10 distinct domains |
| Error types | Decoupled from domain (prevents pattern-matching) |
| Subtlety | Easy / Medium / Hard — varied error positions to detect lazy evaluation |
| Schema | Boolean error flags · categorical descriptors · 1–7 confidence scales · `item_id` for cross-run alignment |

### Scoring

**Tier 1 (Rule-Based):** Binary checks for detection accuracy, localization accuracy, and correction accuracy.

**Tier 2 (Judge-Based):** An LLM judge evaluates analytical quality and step-level logical consistency.

> Prompts use neutral language (e.g., *"Deliberate"*) to avoid biasing models toward error-hunting, accounting for known LLM sensitivity to prompt framing.

---

## Metrics

### Core Performance

| Metric | Definition |
|--------|-----------|
| **Overall Score** | Total Tier 1 + Tier 2 points earned / total possible |
| **Monitoring (Detection)** | Mean match rate of `ERROR_PRESENT`, `ERROR_STEP`, `ERROR_TYPE` |
| **Control (Correction)** | Mean `CORRECTION_match` over error-present items only |
| **False Positive Rate** | Rate of error reports on flawless chains |
| **Error Subtlety Degradation** | Performance gap between easy and hard errors |
| **Logic Consistency** | Judge-verified alignment between stated `ERROR_STEP` and model's `MY_THINKING` block |

### Advanced Diagnostics & Calibration

| Metric | Definition |
|--------|-----------|
| **Retrospective ECE** | Confidence-vs-accuracy calibration *after* reasoning |
| **Prospective ECE** | Pre-task familiarity vs. final accuracy calibration |
| **r_pb** | Point-biserial correlation: familiarity → accuracy rank |
| **Metacognitive Blind Spot Rate** | Items where `ERROR_PRESENT_match = 0` AND confidence ≥ 6 |
| **Appropriate Uncertainty Rate** | Items where `overall_score < 1.0` AND confidence ≤ 3 |
| **Hallucinated Logic Rate** | Correct `CORRECTION_match` but 0 on Step Consistency or Analytical Thinking |
| **Reasoning Length vs. Accuracy** | Point-biserial: `RAW_Thinking_Word_Count` → binarized perfect score |

---

## Results

### Table 1 — Core Performance & Reliability

| Model | Overall Score | Monitoring | Control | False Positive Rate | Subtlety Degradation | Logic Consistency |
|-------|:---:|:---:|:---:|:---:|:---:|:---:|
| gemma-4-31b | **94.91%** | 85.3% | 99.0% | 3.0% | 5.0% | **100.0%** |
| gemini-3-flash-preview | 93.80% | 84.8% | 99.0% | 11.5% | 14.4% | 98.0% |
| gemini-3.1-pro-preview | 91.59% | 84.7% | 99.0% | 6.0% | 8.8% | 98.0% |
| claude-opus-4-6 | 90.10% | 80.8% | 99.0% | 19.0% | 20.9% | 96.8% |
| deepseek-r1-0528 | 89.57% | 74.1% | 89.0% | 39.5% | **0.6%** | 99.0% |
| gpt-oss-20b | 84.96% | 66.7% | 56.0% | 14.0% | 17.2% | 95.0% |
| gpt-5.4-2026-03-05 | 81.35% | 82.6% | 98.0% | 13.5% | 19.6% | 98.0% |
| gemma-3-12b | 79.33% | 65.1% | 73.5% | 24.0% | 4.3% | 86.5% |
| gpt-oss-120b | 76.98% | 66.2% | 65.0% | 5.0% | 7.6% | 99.4% |
| gemma-3-1b | 29.17% | 34.9% | 0.0% | 14.0% | 7.6% | 19.5% |

### Table 2 — Advanced Behavioral Diagnostics & Calibration

| Model | Retro. ECE | Prosp. ECE | r_pb | Blind Spot Rate | Appr. Uncertainty | Hall. Logic | Reasoning Length vs. Accuracy |
|-------|:---:|:---:|:---:|:---:|:---:|:---:|:---:|
| gemma-4-31b | 0.0493 | 0.2192 | +0.133 | 2.0% | 0.0% | **0.0%** | −0.21 *(Confused)* |
| gemini-3-flash-preview | 0.0491 | 0.2487 | +0.028 | 4.0% | 0.0% | 0.5% | −0.17 |
| gemini-3.1-pro-preview | 0.0791 | 0.2450 | NaN | 3.0% | 0.0% | 0.0% | −0.01 |
| claude-opus-4-6 | 0.0946 | 0.2692 | +0.254 | 2.8% | 4.5% | 1.0% | −0.35 *(Confused)* |
| deepseek-r1-0528 | **0.0449** | 0.3623 | +0.025 | 5.8% | **19.8%** | 0.6% | −0.18 |
| gpt-oss-20b | 0.1195 | 0.4745 | +0.028 | 6.0% | 2.0% | 0.0% | +0.02 |
| gpt-5.4-2026-03-05 | 0.1531 | 0.2654 | +0.140 | 6.5% | 0.0% | ⚠️ 80.6% | +0.06 |
| gemma-3-12b | 0.1814 | 0.3037 | +0.076 | 20.5% | 0.2% | 8.8% | +0.09 |
| gpt-oss-120b | **0.0308** | 0.2649 | +0.070 | 2.5% | 0.0% | 1.5% | −0.03 |
| gemma-3-1b | 0.2996 | 0.4010 | −0.150 | 14.8% | 58.5% | 0.0% | +0.00 |

<img width="5970" height="1320" alt="image" src="https://github.com/user-attachments/assets/65c7cd67-55d1-4fd0-891c-2532aace3c1c" />
<img width="5970" height="1320" alt="image" src="https://github.com/user-attachments/assets/564bd5ab-5577-43bb-9765-0639edac96c4" />
<img width="1080" height="608" alt="image" src="https://github.com/user-attachments/assets/d7e32ad4-1132-4018-b04d-aca8fff477b7" />

---

## Key Findings

### 🔲 The Black Box Metacognition Gap
GPT-5.4 achieved 98% repair accuracy (Control) but triggered an **80.6% Hallucinated Logic rate**. Manual review revealed the model explicitly refusing to provide Chain-of-Thought reasoning, stating it *"can't provide private chain-of-thought."* Without externalized thinking, frontier models cannot prove they are monitoring logic — they demonstrate execution capability but not the logical auditing required to verify *why* an answer is wrong. This has direct implications for AI safety, independent of any marketing claims about reasoning transparency.

### 📈 Scaling & the Medium-Model Sweet Spot
Metacognitive ability is non-linear with scale. Gemma 4 31B achieved the top Overall Score (94.91%), outperforming models orders of magnitude larger. Meanwhile, small models (1B and 12B) suffer from total task failure and the highest Blind Spot Rates (up to 20.5%). Metacognition appears to be a **limited emergent capability**, not a smooth function of parameter count.

### 🎯 Calibration Asymmetry
Across all models, Retrospective ECE is significantly better than Prospective ECE — models effectively *don't know what they know* until they begin reasoning. This is a universal failure of prospective self-monitoring. Gemini models exhibit systematic overconfidence; Claude Opus and GPT-5.4 show the only marginal calibration among frontier systems.

### ⚠️ False Positives & Confused Rambling
Elite models show a *negative* correlation between reasoning length and accuracy — generating more tokens when confused rather than confident. DeepSeek R1 is the notable outlier: despite a 39.5% False Positive Rate, it holds the highest Appropriate Uncertainty rate (19.8%), suggesting a distinct epistemic humility profile not captured by accuracy alone.

---

## Conclusion

Metacognitive awareness is unlikely to be present in small models, but it is not guaranteed in large ones. Frontier models dominate execution and repair; medium-sized models currently yield the most reliable objective monitoring. The emergent, non-linear nature of metacognition suggests it is not a default capability that scales automatically.

The broader trend of obfuscating internal reasoning in frontier systems creates a verification problem: high performance is achievable while making logical auditing impossible — a concern that extends beyond benchmark design into AI safety.

---

## Repository Structure

```
ARGUS/
├── README.md
├── Benchmark Script     # Full evaluation notebook
├── Input Data           # Data used to test models  
    ├── task_1_demo_v2.json     # Small dataset to test benchmark functionality
    └── task_1_full_v2          # Full dataset used in testing
├── Test Data            # Result data from benchmark tests
    ├── Claude Opus 4.6 ARGUS_Results_trial_1.xlsx
    ├── Claude Opus 4.6 ARGUS_Results_trial_2.xlsx
    ├── DeepSeek R1 ARGUS_Results_trial_1.xlsx
    ├── DeepSeek R1 ARGUS_Results_trial_2.xlsx
    ├── Gemini 3 Flash Preview ARGUS_Results_trial_1.xlsx
    ├── Gemini 3 Flash Preview ARGUS_Results_trial_2.xlsx
    ├── Gemini 3.1 Pro Preview ARGUS_Results_trial_1.xlsx
    ├── Gemma 3 1b ARGUS_Results_trial_1.xlsx
    ├── Gemma 3 1B ARGUS_Results_trial_2.xlsx
    ├── Gemma 3 12b ARGUS_Results_trial_1.xlsx
    ├── Gemma 3 12B ARGUS_Results_trial_2.xlsx
    ├── Gemma 4 31b ARGUS_Results_trial_1.xlsx
    ├── GPT 5.4 ARGUS_Results_trial_1.xlsx
    ├── GPT 5.4 ARGUS_Results_trial_2.xlsx
    ├── gpt oss 20B ARGUS_Results_trial_2.xlsx
    └── gpt oss 120b ARGUS_Results_trial_1.xlsx     
```

---

## References

- Nelson, T. O., & Narens, L. (1990). Metamemory: A theoretical framework and new findings. *Psychology of Learning and Motivation*, 26, 125–173.
- Flavell, J. H. (1979). Metacognition and cognitive monitoring. *American Psychologist*, 34(10), 906–911.
- MetaMedQA (2025); Zhou et al. (2025); KalshiBench (2025); Ackerman (2025).

---

*ARGUS was submitted to the Google DeepMind × Kaggle AGI Evaluation Hackathon (2026).*
