Architecture — AI Executive Summarizer

Node-by-node walkthrough of workflows/ai-executive-summarizer.json, including the failure paths.

The workflow has one job: take an arbitrary structured payload from an external system and return three decision-ready takeaways, or fail loudly instead of returning something plausible but wrong.

Flow overview
Webhook → Validate → Normalize → LLM → Parse & check → Format → Deliver
                │                        │
                └──── reject (400)       └──── re-prompt → dead-letter
1. Webhook ingestion

Node: Webhook (POST, response mode: last node)

Accepts JSON payloads at /webhook/executive-summary. No authentication is baked into the exported file — add header auth or an n8n credential before exposing the endpoint publicly.

Why response mode matters: the caller receives the finished brief rather than an immediate 200. Slower, but it makes the workflow usable synchronously from a script or a chat command.

2. Schema validation

Node: IF / Code

Rejects the request before any token is spent if:

required keys are missing (source, event)
the payload exceeds the size ceiling
event is not in the allowed set

Design intent. Validation sits upstream of the model, not after it. A small model handles clean input well and ambiguous input badly, so the cheapest reliability gain is refusing to send it ambiguous input. Rejections return 400 and are excluded from the reliability metric by design — they are the guardrail working, not a failure.

3. Normalization

Node: Set / Code

Maps source-specific field names onto one internal contract, so the prompt doesn't have to know whether the event came from HubSpot, a form, or a cron job.

Internal field	Type	Notes
source	string	Origin system
event	string	Event type
metrics	object	Numeric values to reason over
notes	string	Free-text context, truncated

Free text is truncated here rather than in the prompt: bounding input length at the data layer keeps cost predictable regardless of what a sales rep typed.

4. AI layer

Node: OpenAI — model gpt-4o-mini, temperature 0.2

Full system prompt and output contract: prompts/executive-summary.md

Why this model. The task is extraction and compression, not multi-step reasoning. A frontier model produces marginally better prose at many times the cost per run, and prose quality is not the bottleneck — the bottleneck is whether the output is parseable and factually anchored to the payload.

Why low temperature. Executive briefs should be reproducible. The same payload twice should not produce two different recommendations.

5. Parse and validate output

Node: Code (JSON parse in a try/catch) → IF

The model is instructed to return JSON only. This node checks that:

the response parses as JSON
takeaways is an array of exactly three strings
no takeaway is empty or exceeds the length ceiling

Design intent. Downstream nodes never parse free text. Without this branch a malformed response would be written to the database as if it were valid — the failure mode that corrupts data silently instead of alerting anyone.

6. Retry path

Node: OpenAI (second call, stricter prompt)

On a parse failure the payload is re-sent with the schema restated and the offending output included as a negative example.

Retries are bounded. A prompt that keeps failing cannot loop and cannot burn budget unattended.

7. Dead-letter path

Node: Set → storage / alert

Events that exhaust the retry budget are parked with the original payload and the last model output attached, then flagged for manual review.

Design intent. The alternative — forcing a partial result through — trades a visible failure for an invisible one. A brief that quietly omits the worst number is more dangerous than no brief at all.

8. Formatting and delivery

Node: Code → Slack / Notion / HTTP

Renders the validated JSON as Markdown and pushes it downstream. Delivery is the only node most people need to change when adapting the blueprint.

Idempotency

Repeated deliveries of the same event (a common webhook behaviour on retry) are deduplicated on an event ID before the model is called. Without this, one upstream hiccup produces two contradictory briefs on the same numbers.

Security notes
No credentials are stored in the workflow JSON. All keys live in n8n's credential store, which is why the exported file is safe to publish.
The webhook path is configurable via N8N_WEBHOOK_PATH; don't reuse the example value in production.
Payload contents are sent to the OpenAI API. Don't route personal data through this workflow without checking your own legal basis first.
