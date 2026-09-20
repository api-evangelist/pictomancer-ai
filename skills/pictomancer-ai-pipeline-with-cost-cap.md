---
generated: '2026-09-19'
method: generated
name: Run a multi-step pipeline under a cost cap
description: Chain resize, compress, convert and crop in one request, priced first and capped with X-Max-Cost-USD.
api: openapi/pictomancer-ai-openapi.yml
operations: [analyze_image, estimate_cost, image_pipeline]
mcp_tools: [analyze_image, estimate_cost, image_pipeline]
source: operationIds verified in openapi/pictomancer-ai-openapi.yml; PipelineRequest/PipelineOperation/EstimateRequest schemas; homepage receipt example
---

# Run a multi-step pipeline under a cost cap

A pipeline fetches the image once, applies up to 10 operations in order, and is billed as ONE operation.

## Steps
1. **`analyze_image`** — `POST /v1/analyze` `{"source": "..."}`. Free. Take `size_bytes`.
2. **`estimate_cost`** — `POST /v1/estimate` `{"operation": "pipeline", "input_bytes": <size_bytes>, "operations": [{"type": "resize"}, {"type": "convert", "format": "webp"}]}`. Free. Read `price_usd`, `size_multiplier`, `pipeline_discount`, `free_tier_remaining`.
3. **`image_pipeline`** — `POST /v1/pipeline` with header `X-Max-Cost-USD: <price_usd>`:
   ```json
   {"source": "https://example.com/in.jpg",
    "operations": [{"type": "resize",  "params": {"scale": "0.625"}},
                   {"type": "convert", "params": {"format": "webp", "q": "80"}}]}
   ```
   `params` values are strings. Allowed `type` values: resize, compress, convert, crop. Optional `delivery` (inline | put_url | callback_url).

## What the homepage receipt shows
- A 2.9 MB JPEG (3072x2048) -> resize x0.625 -> convert webp q=80 -> 439 KB, 85.1% saved, billed $0.0045 ($0.003 base x 1.5 size multiplier for a 1-5 MB input).

## Rules
- 412 = your cap was lower than the computed price; 402 = free tier exhausted, pay via x402 or use an API key; 422 = schema error (`errors/pictomancer-ai-problem-types.yml`).
- Max 10 operations; 120 requests/minute per identity (`rate-limits/pictomancer-ai-rate-limits.yml`).
- Nothing to roll back: the API holds no state and deletes inputs within 24h; the charge itself is non-refundable (`conventions/pictomancer-ai-conventions.yml` reversibility block).
