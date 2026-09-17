# Ausca Document Analysis: one paid run, end to end

- **Discovery — 220 ms timed re-fetch.** I read `https://ausca.com/skills/document-analysis/SKILL.md` and `https://ausca.com/catalog.json`, then bound the request to `document.analysis`, revision `analysis-bytes-r5`, revision digest `sha256:cd8234...f6fa2`, input schema `sha256:709321...6759b`, output schema `sha256:f42f96...b8a55`, and canonicalizer `runx.receipt.c14n.v1`. The first fetch and the later timed re-fetch used the same two surfaces.

- **Artifact commit — 2,159 ms.** I made a real ledger snapshot PNG, 900×1500 and 130,997 bytes, containing the enterprise's actual capital and funnel figures. `POST /v1/artifacts` returned HTTP 200 and `runx:artifact:sha256:0ab50d...c7d6d`; its returned content digest exactly matched the local file. I selected `FORMS`, `TABLES`, and `LAYOUT`.

- **Unsigned challenge — 5,193 ms.** The first `POST /v1/analyze-document` had no payment header and returned HTTP 402. `PAYMENT-REQUIRED` decoded as x402 v2, 300000 units of USDC at `0x833589...2913` on `eip155:8453`, payable to `0x26572f...b422`. The JSON body only carried `code=payment_required`; the terms lived in the header.

- **Issue 1, at the challenge step.** The 402's `extensions.bazaar.info.input.body` advertises `runner=analysis-submit` with an `inputs` object. That is not the `InvocationEnvelope` accepted by this route and does not contain the catalog bindings the skill requires. A generic x402 discovery client following that example would construct the wrong body.

- **Payment and admission — 3,624 ms.** I retried the exact same request bytes, SHA-256 `de2cba...29e5`, with `PAYMENT-SIGNATURE`. The response was HTTP 202 with `status=accepted`, `state=admitted`, and invocation `paid_91e461ca-14bb-4b00-9e36-8cf20880f9ee`. The x402 response named transaction `0xfbf7c2...45f8`; Base later returned transaction status 1 in block 51428973. The wallet fell from 1.01 to 0.71 USDC, exactly the quoted 0.30.

- **Asynchronous wait — 27,800 ms.** I polled `GET /v1/invocations/paid_91e461ca-14bb-4b00-9e36-8cf20880f9ee` six times. The first five reads were `running`; the sixth was `succeeded`. The server recorded admission at `12:14:51.342Z` and success at `12:15:20.970Z`.

- **Result retrieval — 1,953 ms.** The terminal manifest referenced page artifact `runx:artifact:sha256:b3fcdf...fdad`. I minted a download URL with `POST /v1/artifacts/{artifact_ref}/access`, downloaded 2,956 bytes, and reproduced page digest `sha256:26adfe...8c61`.

- **Issue 2, at result retrieval.** The OpenAPI component `CreateArtifactAccessInput` requires `artifact_ref` and `idempotency_key` as if they were one input object, while the HTTP operation actually binds the former in the path and the latter in the `Idempotency-Key` header. The endpoint description is usable, but the component and transport shape pull in different directions for generated clients.

- **Validation — under 1 ms locally.** The inline manifest passed the published `document-analysis.output.schema.json`; the raw schema file itself reproduced catalog digest `sha256:f42f96...b8a55`. Compact canonical JSON of the output reproduced `output_digest=sha256:3975e5...3027`, and the referenced page bytes reproduced their own digest. The extracted text included both tables and every numeric value in the source.

- **Receipt — 594 ms.** `https://runx.ai/r/e073e971022e1e41f74f27497a65be4caf41834082d1b69a85826e78d2427b32` returned HTTP 200. It says “Ausca Document Analysis completed” and dates the notarization to 12:15 PM, which fits the invocation timeline.

- **Issue 3, at receipt inspection.** The public page calls the receipt “L1 Notarized” and “hash only”, says the signing key is unclaimed, and does not show `offer_id=document.analysis`, the 0.30 USDC amount, payer, transaction, or Base network. That conflicts with the review instruction to confirm the offer and settled amount on the page. I could confirm those facts only by combining the terminal invocation, x402 response, and chain receipt.

- **Issue 4, at machine-readable receipt inspection — 262 ms.** The page advertises an `application/json` alternate URL at `https://runx.ai/v1/receipts/notarizations/e073e971...`, but that URL returned HTTP 404. This is a concrete dead link, not a presentation preference.

- **Concrete change.** Turn the public receipt into a self-contained verification page: show the offer id, amount/asset/network, payer, settlement transaction, and a claimed Ausca notary identity, then serve the same verified fields at the advertised JSON URL. This would make the stated review gate possible from the receipt alone and remove both receipt-step failures.

- **Total run time — 84,402 ms.** This is measured from artifact-upload start at `12:13:56.568Z` through terminal success at `12:15:20.970Z`. It includes the deliberate unsigned challenge, challenge inspection, signing, settlement, admission, and async wait; later validation, receipt reading, and writing this report are outside that number.

No secret appears here. The private key never left the local signer, and the actual `PAYMENT-SIGNATURE` is represented as `<redacted-payment-signature>` in the JSON evidence. The full request/response identifiers, timings, digests, and public transaction are in [evidence.json](./evidence.json).
