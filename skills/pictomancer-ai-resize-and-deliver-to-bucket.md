---
generated: '2026-09-19'
method: generated
name: Resize and deliver straight to your bucket
description: Resize an image and have the result PUT to your own S3-compatible storage without the provider ever holding your credentials.
api: openapi/pictomancer-ai-openapi.yml
operations: [get_format_info, estimate_cost, resize_image]
mcp_tools: [get_format_info, estimate_cost, resize_image]
source: operationIds verified in openapi/pictomancer-ai-openapi.yml; delivery semantics from components.schemas.PutUrlDelivery, llms.txt "Delivery (output target)" and changelog v0.6.0
---

# Resize and deliver straight to your bucket

## Steps
1. **`get_format_info`** — `GET /v1/info`. Free. Confirms output formats (jpeg, png, webp, tiff, gif, avif) and each format's options (`q`, `strip`, `lossless`, `effort`, ...).
2. **Sign a presigned HTTPS PUT URL** on your side (S3, R2, GCS, Azure, B2, DO Spaces). The provider only uses the URL and discards it.
3. **`estimate_cost`** — `POST /v1/estimate` `{"operation": "resize", "input_bytes": <bytes>}`. Free; returns `price_usd`.
4. **`resize_image`** — `POST /v1/resize`:
   ```json
   {"source": "https://example.com/in.jpg", "scale": 0.5, "format": "webp",
    "delivery": {"mode": "put_url", "put_url": "https://bucket.s3.amazonaws.com/key?X-Amz-Signature=...",
                 "headers": {"Content-Type": "image/webp", "Cache-Control": "public, max-age=31536000"}}}
   ```
   Use `scale` (uniform), `scale_x`/`scale_y`, or `width`+`height` for fill mode with `gravity` (attention | entropy | centre) — the modes are mutually exclusive. Optional modifiers `autorot`, `denoise` (1-3), `equalize`, `sharpen` cost nothing extra.
5. **Read the JSON response** `{etag, status, bytes_written, duration_ms, content_type}` — with `put_url` delivery the body is JSON, not image bytes.

## Rules
- `put_url` must be https; internal IP ranges are blocked; only whitelisted storage headers pass (Content-Type, Cache-Control, x-amz-acl, x-amz-server-side-encryption, x-ms-blob-content-type, x-goog-meta-*).
- Add `X-Max-Cost-USD` to cap the charge (412 on overrun). Identity/payment as in `authentication/pictomancer-ai-authentication.yml`.
- Replays are billed again — there is no idempotency key (`conventions/pictomancer-ai-conventions.yml`).
