# Benchmarks
 
Raw measurement data behind the Results table in the [README](../README.md).
 
> [!WARNING]
> **This file is a template. It is not yet filled with measured data.**
> Until you replace every placeholder below, remove the Results table from the README or relabel its figures as estimates. An empty benchmarks file linked from a metrics table is worse than no file at all: it tells a reader the numbers were never measured.
>
> Delete this warning block once the data is in.
 
---
 
## Environment
 
| Field | Value |
|:--|:--|
| n8n version | `<version>` |
| Hosting | `<self-hosted / n8n cloud>` |
| Model | `gpt-4o-mini` |
| Temperature | `0.2` |
| Measurement period | `<start date> – <end date>` |
| Sample size | `<n>` executions |
 
---
 
## 1. Time to insight
 
**Baseline (manual).** Time from opening the source dashboard to a written three-point brief, measured with a stopwatch.
 
| Run | Date | Duration |
|:-:|:--|:--|
| 1 | | |
| 2 | | |
| 3 | | |
| | **Median** | |
 
**Automated.** End-to-end execution duration from the n8n execution log (`Executions` → select run → duration).
 
| Run | Date | Duration |
|:-:|:--|:--|
| 1 | | |
| 2 | | |
| 3 | | |
| | **Median** | |
 
**Result:** `<baseline median> → <automated median>`
 
---
 
## 2. Cost per run
 
Token counts from the OpenAI usage dashboard, same payload and same prompt across both models.
 
| Model | Input tokens | Output tokens | Cost per run |
|:--|--:|--:|--:|
| `<frontier model used in the prototype>` | | | |
| `gpt-4o-mini` | | | |
| | | **Reduction** | `<%>` |
 
**Note.** State which frontier model the comparison is against. "92% cheaper than standard models" is not checkable; "92% cheaper than `<model>` at identical prompt and output length" is.
 
---
 
## 3. Reliability
 
From the n8n execution log, filtered to schema-valid payloads.
 
| Outcome | Count |
|:--|--:|
| Succeeded on first pass | |
| Succeeded after re-prompt | |
| Sent to dead-letter | |
| **Total executions** | |
| **Success rate** | `<%>` |
 
Rejected malformed payloads are excluded — they never reach the model, so counting them would measure the validator, not the workflow.
 
---
 
## Failure log
 
Every dead-lettered event, with cause. This section is the one a senior engineer reads first: a benchmark file with no failures listed reads as untested, not as flawless.
 
| Date | Event | Cause | Fix applied |
|:--|:--|:--|:--|
| | | | |
 
---
 
## How to reproduce
 
1. Import the blueprint into a clean n8n instance.
2. Fire the payloads in [`prompts/executive-summary.md`](../prompts/executive-summary.md) against the webhook.
3. Read durations from `Executions`, token counts from the OpenAI usage dashboard.
4. Report the median, not the best run.
 
