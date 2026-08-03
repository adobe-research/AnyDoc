# Prompts

This directory contains the model prompts used throughout AnyDoc, transcribed
from the paper's supplementary material. Variable fields are marked with
`{...}` placeholders to be filled in at inference time.

| File | Used for |
| --- | --- |
| [`01_intention_and_description_annotation.txt`](01_intention_and_description_annotation.txt) | Semantic annotation of source documents (intention + description) with InternVL3, during DocHTML construction. |
| [`02_html_css_code_generation.txt`](02_html_css_code_generation.txt) | HTML/CSS document synthesis with Qwen3-Coder-480B, during DocHTML construction. |
| [`03_baseline_task_instructions.txt`](03_baseline_task_instructions.txt) | Task-specific instructions appended to the generation prompt for the zero-shot GPT-4o / InternVL3-78B baselines (I2D, DD, E2D). |
| [`04_flux_i2d_baseline.txt`](04_flux_i2d_baseline.txt) | Text prompt for the FLUX.1-dev intention-to-document baseline. |

> **Note.** This repository does not release training or inference code — only
> the prompts above are provided as plain text for reference and reproducibility.
