---
generated: '2026-09-19'
method: generated
name: Ship an AI-generated image web-ready
description: Take the 2-8 MB PNG a generator returned and get back a web-ready WebP/AVIF, knowing the price first.
api: openapi/pictomancer-ai-openapi.yml
operations: [analyze_image, estimate_cost, optimize_generated_image]
mcp_tools: [analyze_image, estimate_cost, optimize_generated_image]
source: operationIds verified in openapi/pictomancer-ai-openapi.yml; behaviour from the operation descriptions, llms.txt and https://pictomancer.ai/for-agents/ai-generated-images
---

# Ship an AI-generated image web-ready

The second call after gpt-image / DALL-E / Flux / Midjourney / Stable Diffusion: same picture, 5-15x smaller, transparency kept, metadata stripped, never upscaled.

## Identity and payment
- No account needed: the first 50 requests per identity are free. Send `X-Agent-Wallet: <ethereum address>` to hold a stable identity (otherwise the IP is used). See `authentication/pictomancer-ai-authentication.yml`.
- After the free tier the API answers **402** with an x402 price (USDC on Base); pay and retry with `X-Payment`, or send `Authorization: Bearer <api key>`.

## Steps
1. **`analyze_image`** — `POST /v1/analyze` `{"source": "<png url or base64>"}`. Free. Read `size_bytes`, `width`, `height`, `format`, and `provenance.c2pa` (a manifest will not survive re-encoding).
2. **`estimate_cost`** — `POST /v1/estimate` `{"operation": "optimize_generated", "input_bytes": <size_bytes>, "format": "webp"}`. Free. Read `price_usd` and `within_free_tier`.
3. **`optimize_generated_image`** — `POST /v1/optimize_generated` `{"source": "...", "format": "webp", "max_dimension": 1600}`; add `X-Max-Cost-USD: <price_usd>` so the API refuses with **412** rather than charging more. Optional `q` or `quality_target` (SSIM, +$0.004), `strip` (default true). Bytes come back inline, or set `delivery` to `put_url` / `callback_url`.

## Read the receipt
- `X-Pig-Billed` is `0` when the output was not smaller — the result is still returned and no free-tier slot is spent.
- `X-Pictomancer-Bytes-Before`, `-After`, `-Saved-Percent` report the saving; `X-Pictomancer-C2PA-Input` says whether the input carried Content Credentials.

## Rules
- Price = base ($0.002) x size multiplier (1.5x from 1 MB, 2x from 5 MB) + surcharges (avif +$0.001). See `plans/pictomancer-ai-plans-pricing.yml`.
- No idempotency key exists; a retry is billed again. Keep your own ledger (`conventions/pictomancer-ai-conventions.yml`).
- 422 means a schema error — read `detail[].loc` (`errors/pictomancer-ai-problem-types.yml`). Stay under 120 requests/minute (`X-RateLimit-Remaining`).
