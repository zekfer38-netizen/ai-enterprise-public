# Ausca Media Transcription — independent run

- **Discovery (about 0.34 s).** I resolved `media.transcription` from the live catalog, rather than copying hashes from prose. The active revision was `transcription-bytes-r6`.
- **Artifact commitment (about 2.07 s).** I generated a real 11.03-second English MP3, 88,320 bytes, and committed it through `POST /v1/artifacts`. The returned artifact reference and SHA-256 matched the local bytes.
- **Challenge (about 3.84 s).** The unsigned `POST /v1/transcribe-media` returned HTTP 402. The body only said `payment_required`; the actual x402 v2 terms were in `PAYMENT-REQUIRED`: 400000 atomic USDC on `eip155:8453` to `0x26572ff23c6c52bfb1a69cb0c9114a8be443b422`.
- **Payment (about 6.34 s).** I signed the exact same JSON and paid once. `PAYMENT-RESPONSE` reported payer `0x8A17AffDD899e5C1232C534CF1E4DD05d8eD27AD`, transaction `0x0aa374d29a6e658ce923f2a657d053b6623b88b782b6ca517518f29004b31c67`, and success. Base returned receipt status `0x1`.
- **Admission (about 0.23 s).** The paid replay returned HTTP 202 with `state=admitted`, invocation `paid_e51cd1fd-787f-49eb-8a57-7cbd7332ab66`, and a relative `inspect_url`.
- **Asynchronous wait (about 28.6 s).** The response advertised `Retry-After: 2`. Fourteen polls reached `state=succeeded`; no second payment or replay was needed.
- **Result validation (about 1.8 s).** The result contained one segment from 0 to 11.03 seconds and the exact spoken sentence. It passed the published output schema, and `source_digest` matched the committed MP3 digest.
- **Receipt (about 1.7 s).** The public receipt and its JSON endpoint returned HTTP 200. They identify Ausca Media Transcription and declare 0.40 USD, but classify the proof as `L1 Notarized` and `hash-only`; the page does not independently prove the underlying payment.

## Concrete findings

1. **Payment terms are header-only on HTTP 402.** The body from `POST /v1/transcribe-media` is `{"code":"payment_required","message":"Payment authority is required.","status":"error"}`. A non-browser client must decode the base64 `PAYMENT-REQUIRED` header to learn amount, asset, network, and recipient. A compact copy of those terms in the JSON body would make the failure self-describing.
2. **The live `bazaar` extension contains discovery placeholders.** The 402 header includes `idempotency_key=ausca-x402-discovery-example` and fake artifact digests inside `extensions.bazaar.info.input.body`, while the real request used a different key and artifact. Marking this block explicitly as example-only, or removing it from the live challenge, would reduce the chance of a client copying placeholders.
3. **The receipt is useful but deliberately limited.** `https://runx.ai/r/82097c38c1158fd924612a8947c5eff6b11f43b3b76f6c797cb8ddb423b42691` says `hash-only`, `edge_key.identity_status=unclaimed`, and does not publish the transaction identifier. A linked payment receipt or an explicit pointer to `PAYMENT-RESPONSE` would make the settlement trail easier to audit.

## Suggested change

Keep the current 402 header contract, but add a small machine-readable `payment` object to the error body and label the Bazaar example as non-executable. Also expose a durable settlement-reference link alongside the public service receipt.
