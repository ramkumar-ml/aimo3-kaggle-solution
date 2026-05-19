# AIMO3 Kaggle Solution Notes

This repository accompanies my Substack article about spending the final 45 days of the Kaggle **AI Mathematical Olympiad Progress Prize 3 (AIMO3)** competition trying to understand, evaluate, and improve a public notebook built around `gpt-oss-120b`.

The main theme of the project is straightforward:

- Start from a strong public `gpt-oss-120b` notebook already solving many problems.
- Understand its inference architecture end to end instead of treating it as a black box.
- Try alternate directions such as smaller models, speculative decoding, and fine-tuning.
- Return to the original setup and make targeted speed and reliability improvements under Kaggle's 5-hour / single-GPU constraint.

The article describes the reasoning and lessons learned. This repository contains the actual notebooks behind those experiments.

## Competition Context

The competition required a Kaggle notebook to solve **50 olympiad-style math problems** within a **5-hour budget** on a **single GPU**. In practice, most of the work here assumes Kaggle's H100 environment, local vLLM serving, and tool-using reasoning loops where the model can write and run Python during problem solving.

My final results:

- Joined late, with about 45 days left.
- Best public leaderboard score: **39/50**
- Final private score: **40/50**
- Final rank: **1897 / 3452**

## Repository Structure

```text
notebooks/
  01_famous_public_notebook.ipynb
  02_qwq_experiment.ipynb
  03_nemotron_experiment.ipynb
  04_speculative_decoding_experiment.ipynb
  05_finetuning_data_pipeline.ipynb
  06_final_submission.ipynb
README.md
```

## Notebook Guide

### 01. `01_famous_public_notebook.ipynb`

This is the baseline public notebook that shaped the rest of the work.

What it does:

- Runs `gpt-oss-120b` through a local vLLM server.
- Solves each problem with multiple parallel attempts.
- Allows the model to call Python tools in a separate sandboxed kernel.
- Uses agreement between attempts plus entropy-based confidence selection to choose a final answer.
- Manages time across the full problem set and can run against Kaggle test data.

Why it matters:

- This notebook is the foundation for understanding the competition setup.
- Most later changes are easier to understand by comparing them against this baseline.

### 02. `02_qwq_experiment.ipynb`

This notebook evaluates **QwQ-32B** as a possible alternative to `gpt-oss-120b`.

What it tests:

- Whether a smaller math-focused model could free enough GPU memory to improve throughput.
- Whether the overall notebook architecture could be reused with a different serving configuration.

Conclusion from the experiment:

- QwQ produced too much reasoning text and each attempt took too long.
- It was not practical under the 5-hour competition budget.

### 03. `03_nemotron_experiment.ipynb`

This notebook tests **NVIDIA Nemotron-Super-120B** inside the same broad inference framework.

What it explores:

- Whether Nemotron's claimed throughput and math ability could outperform the baseline on Kaggle hardware.
- Logging and instrumentation around progress and GPU usage.

Conclusion from the experiment:

- On Kaggle H100, this setup was slower than expected.
- The article explains the likely reason: the model's quantization format was a poor fit for the hardware path used in practice.

### 04. `04_speculative_decoding_experiment.ipynb`

This notebook tries **speculative decoding** with `gpt-oss-120b` plus a draft model.

What it changes:

- Adds speculative decoding configuration and related serving changes.
- Includes logic for truncating long tool outputs more carefully.
- Retains the same overall solver structure as the main inference notebooks.

Conclusion from the experiment:

- The draft model did not agree often enough with the main model to produce a meaningful speedup.
- Extra memory pressure and added complexity outweighed the benefit.

### 05. `05_finetuning_data_pipeline.ipynb`

This notebook is different from the others: it is a **data preparation pipeline** for possible fine-tuning work rather than a competition inference notebook.

What it covers:

- Google Drive mounting and Colab-oriented workflow.
- Hugging Face login via Colab secrets.
- Streaming large datasets instead of loading them fully into RAM.
- Filtering, leakage prevention, curriculum ordering, sampling, packing, and validation.
- Saving a curated fine-tuning dataset.

Why it exists:

- I explored fine-tuning seriously before deciding that throughput and inference efficiency were the bigger bottlenecks for this competition.

### 06. `06_final_submission.ipynb`

This is the final inference notebook derived from the baseline after experimentation.

It reflects the direction that survived the failed branches:

- Stay with `gpt-oss-120b`.
- Keep the multi-attempt tool-calling structure.
- Apply targeted improvements that help under the Kaggle time budget.

The article highlights two changes that mattered most:

- **Prefix caching fix**: move shared tool instructions before the problem text so vLLM can reuse the common prefix across requests.
- **Output truncation fix**: keep the beginning and end of long tool outputs instead of only the beginning, preserving the most useful result information while reducing context growth.

