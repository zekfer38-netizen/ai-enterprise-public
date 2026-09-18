# Ausca Agent Inbox: paid create, real mail, verified deletion

- **Discovery — 456 ms timed re-fetch.** I read `https://ausca.com/.well-known/x402`, searched `GET /v1/offers?q=agent%20inbox`, and pinned `GET /v1/offers/inbox.receive` to revision digest `sha256:a1f122...8753`. The selected bindings were `inbox.receive`, revision `receive-duration-r3`, input schema `sha256:a97f97...43be`, and output schema `sha256:b7435b...d63c6`.

- **Issue 1, at discovery.** The root `/.well-known/x402` document describes its service as “Extract normalized text from a scanned document.” It lists `/v1/open-inbox` later, but a client starting from the advertised description is pointed at Document OCR, not the broader Ausca catalog.

- **Issue 2, at exact offer resolution.** `GET /v1/offers/inbox.receive?revision_digest=...` returns the identifiers and digests but no HTTP route and no `input_schema.public_path` or `output_schema.public_path`. I had to combine this response with `catalog.json` or OpenAPI to discover `/v1/open-inbox` and the schema URLs. Adding those three links would make the exact-revision response self-sufficient.

- **Unsigned challenge — 235 ms.** I sent the documented invocation envelope to `POST /v1/open-inbox` with `duration_seconds=3600` and no payment header. HTTP 402 carried x402 v2 terms for 50000 atomic units of USDC at `0x833589...2913` on `eip155:8453`, payable to `0x26572f...b422`. The terms matched the documented 0.05 USDC price.

- **Payment and execution — 4,079 ms by server timestamps.** I signed the live challenge locally and retried the exact same body bytes, SHA-256 `ed6fa3...4416`, with `PAYMENT-SIGNATURE`. Base transaction `0xccda08...9f39` has status 1 in block 51461134 and transfers exactly 0.05 USDC from `0x8A17...27AD` to the quoted recipient. The resource was created at `06:06:55.873Z`; invocation `paid_f88a1305-5530-46a5-9f89-b0750eef76be` was `succeeded` at `06:06:59.952Z`. The paid response was already terminal, so polling count was zero.

- **Client recovery — 976 ms, no second charge.** My first verifier tried to read `receipt_ref` from the paid response before the authoritative invocation read and raised `KeyError`; this was my bug, not an Ausca failure. The capability had existed only in process memory. I replayed the same request body and idempotency key through the documented x402 path. It returned the same invocation and resource access; the wallet was 0.66 USDC both before and after the replay. No second inbox and no second payment were created.

- **Status and controlled send.** Authorized `GET /v1/agent-inboxes/{id}` took 183 ms and returned `active` with the exact one-hour expiry. I then sent one plain-text message from an account I control. Ausca recorded it at `06:07:03.323Z`, at most 7,450 ms after inbox creation. The actual random address, capability, and extension authority are omitted from both artifacts.

- **List and read — 252 ms plus 178 ms.** One authorized `GET /messages` returned one message. `GET /messages/{message_id}` returned the expected subject and unique body token with `attachment_count=0`. This verified a real inbound delivery without rendering sender HTML or touching an attachment.

- **Output validation — 407 ms authoritative read.** The invocation output passed the published `agent-inbox.output.schema.json`. The raw schema bytes reproduced catalog digest `sha256:b7435b...d63c6`, and compact JSON of the private output reproduced `output_digest=sha256:d36454...9a95`. Inbox and message identifiers are represented by SHA-256 digests in the public evidence.

- **Receipt — 351 ms.** `https://runx.ai/r/54e383b06045239f6aed302b5aee030dd165eae4b2f93609de8131586cc72fcb` returned HTTP 200 and says “Ausca Agent Inbox completed,” with a Sep 18, 2026 6:08 AM notarization time that fits the run.

- **Issue 3, at receipt inspection.** The public page labels the proof “L1 Notarized” and “hash only,” but it does not show the 0.05 USDC amount, asset, Base network, payer, or transaction. The bounty asks the reviewer to confirm the settled amount on this page; I could confirm it only by combining the x402 response with the Base transaction receipt.

- **Issue 4, at machine-readable receipt inspection.** The receipt page advertises an `application/json` alternate URL at `https://runx.ai/v1/receipts/notarizations/54e383b06045...`, but that URL returned HTTP 404. This is a dead machine-readable link, not a display preference.

- **Deletion — 408 ms.** `DELETE /v1/agent-inboxes/{id}` returned HTTP 200 with `status=deleted` in 213 ms. A separate authorized status read took 195 ms and returned terminal state `deleted`. The workflow did not leave a live inbox behind.

- **Concrete change.** Give the root x402 document a service-neutral description; add route and schema URLs to the exact-offer response; and turn the receipt into a self-contained proof that shows service, amount, asset, network, payer, transaction, and completion time at both its HTML and advertised JSON URLs.

- **Total workflow — 54,250 ms.** This conservative measurement runs from the Base settlement block at `06:06:55Z` through independently verified deletion at `06:07:49.250Z`. It includes the client recovery. The result was one paid hour, one delivered and read email, one deleted inbox, and exactly one 0.05 USDC charge.

No secret appears here. The private key stayed inside the local signer; `PAYMENT-SIGNATURE`, the random inbox address, the mailbox capability, and the extension authority are replaced with withheld or redacted values in [evidence.json](./evidence.json).
