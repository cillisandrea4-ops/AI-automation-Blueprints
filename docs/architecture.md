# Architecture

Node-by-node walkthrough of the AI Executive Summarizer.

## 1. Webhook ingestion
Receives POST payloads. Requests failing schema validation are rejected before reaching the model.

## 2. Normalization
Maps source-specific field names to the internal contract. Applies guardrails on payload size.

## 3. AI layer
`gpt-4o-mini` with a structured prompt. Full prompt: [`prompts/executive-summary.md`](../prompts/executive-summary.md)

## 4. Validation branch
Output is parsed as JSON. Malformed responses trigger a strict re-prompt, bounded to N retries.

## 5. Dead-letter path
Events exhausting the retry budget are parked for manual review rather than force-processed.

## 6. Delivery
Markdown brief pushed downstream (Slack, Notion, database).
