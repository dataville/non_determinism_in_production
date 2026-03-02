# Non-Determinism in Production: What It Is and How to Measure It

Charles Rice · March 2, 2026

A 30-minute technical talk for an AI accelerator audience covering why large language models produce variable outputs and how to measure that variability systematically using evaluation frameworks.

---

## Repository contents

| File | Description |
|---|---|
| `demo_01_variance.ipynb` | Output variance demonstration — the same prompt sent five times under three temperature conditions |
| `demo_02_evaluation.ipynb` | Evaluation framework demonstration — Qwen 2.5 14B as the product model, Mistral 7B as the judge |
| `ai_talk_env.yaml` | Conda environment definition |
| `slides.pdf` | Presentation slides |

---

## Requirements

### Hardware

Both demos run on Apple Silicon. Demo 2 loads two models simultaneously (14B + 7B at 4-bit quantisation). 32GB unified memory is recommended.

### Software

Create and activate the conda environment:

```bash
conda env create -f ai_talk_env.yaml
conda activate ai_talk_env
```

### Credentials

Two tokens are required. Add them to a credentials file and source it before launching Jupyter:

```bash
# ai_talk.creds — do not commit this file
export GITHUB_TOKEN=your_token_here
export HF_TOKEN=hf_your_token_here
```

```bash
source ai_talk.creds
jupyter lab
```

**GitHub token** — required for Demo 1 (Llama 3.3 70B via GitHub Models API)
Get one at: github.com/settings/tokens → Fine-grained token → Permissions → Models: Read

**HuggingFace token** — required for Demo 2 (MLX-LM fetches model weights from HF Hub)
Get one at: huggingface.co/settings/tokens → New token → Read

---

## Demo 1 — Output variance (`demo_01_variance.ipynb`)

Sends the same prompt to Llama 3.3 70B five times under three conditions:

| Experiment | Temperature | What it shows |
|---|---|---|
| 1 | Default (unset) | How most production systems actually run |
| 2 | 1.0 (explicit) | Full sampling diversity |
| 3 | 0.0 (explicit) | The setting commonly believed to eliminate variance |

The notebook was pre-run before the talk. Output cells contain the real results.

Provider: GitHub Models — free for all GitHub accounts, no credit card required.

---

## Demo 2 — Evaluation framework (`demo_02_evaluation.ipynb`)

Two models with deliberately separate roles:

| Role | Model | How it runs |
|---|---|---|
| Product | Qwen 2.5 14B Instruct (4-bit) | Local via MLX-LM |
| Judge | Mistral 7B Instruct v0.3 (4-bit) | Local via MLX-LM |

The product model answers five test questions. The judge scores each answer against defined criteria using DeepEval's GEval metric with a correctness threshold of 0.7.

The notebook was pre-run before the talk. Output cells contain the real results.

### Test case design

The five questions are designed to produce a realistic mix of results — not all pass, not all fail.

| Q | Topic | Expected outcome | Reason |
|---|---|---|---|
| 1 | Why temperature=0 doesn't guarantee determinism | Pass | Conceptual — general ML knowledge |
| 2 | Atil et al. 2024 — maximum accuracy variation (arXiv:2408.04667) | Fail | Specific figure from a specific paper — hallucination expected |
| 3 | Why the judge must be independent of the product model | Fail | See note below |
| 4 | Wang et al. 2024 — consistency rate (npj Digital Medicine) | Fail | Specific figure from a specific paper — hallucination expected |
| 5 | Scenario: undetected model update, two-sentence format | Borderline | Conceptual reasoning plus format compliance |

**Q3 note.** The model's answer to Q3 is substantively correct — it identifies bias and the need for objectivity. The evaluation scored it as a failure because the expected output criteria was written too narrowly. This is intentional. It demonstrates a real problem in evaluation design: poorly specified criteria produce misleading results. Defining what "correct" means before building an evaluation system is not optional.

### Why the models are kept separate

A model scoring its own outputs will be lenient on its own errors because those errors are consistent with its own reasoning patterns. An independent judge produces a more objective score. The product and judge models in this demo are from different families for this reason.

---

## Research references

Atil, B. et al. (2024). *LLM Inference Inconsistencies.* arXiv:2408.04667.

Wang, S. et al. (2024). Consistency study of LLM prompting strategies. *npj Digital Medicine.*
