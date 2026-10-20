# aurum-verify

**Independent, evidence-backed verification of claims and clean content extraction —
sold per call to autonomous agents over x402. No account, no API key, no human in the loop.**

Two tools, one gate, one receipt, one price.

**Verification.** Send a list of claims and the sources you want them checked against.
Get back a machine-readable verdict for each claim, and behind every surviving verdict a
quote that was checked **character-for-character** against the cited source.

**Extraction.** Send one URL. Get back title, byline, excerpt, word count, and a clean
markdown body with nav, footer, and hidden text stripped. Hidden content — comments,
`display:none`, zero-width characters — is reported as data, never executed.

## Price

**$0.05 USD per call** (50,000 atomic USDC), settled over x402 (exact scheme, USDC on
Base). No subscription, no account, no API key, no minimum.

## Call it

```
POST /v1/verify/claims
Content-Type: application/json

{
  "claims": ["The library was released in 2019."],
  "sources": [{"url": "https://example.org/release-notes"}],
  "require_verbatim_quotes": true
}

-> 402  PAYMENT-REQUIRED: <x402 v1 challenge>
-> 200  PAYMENT-RESPONSE: <x402 v1 settlement>   (with PAYMENT-SIGNATURE credential)
```

Headers follow the canonical x402 v1 family: `PAYMENT-REQUIRED`, `PAYMENT-SIGNATURE`,
`PAYMENT-RESPONSE`. A nonce buys exactly one answer; replay is `402`. An
`idempotency_key` replays the same question for free.

## Extract content

```
POST /v1/extract
Content-Type: application/json

{ "url": "https://example.org/some-article" }

-> 402  PAYMENT-REQUIRED: <x402 v1 challenge>
-> 200  { "article": { "title": ..., "byline": ..., "markdown": ... }, "receipt": ... }
```

One URL in, clean content out. A refused URL (SSRF guard) or a failed fetch returns
`400` and **does not bill**.

## Verdicts

- **supported** — a quotable source states the claim.
- **contradicted** — a quotable source states the opposite.
- **mixed** — sources disagree with each other.
- **unverifiable** — no quotable source found. A valid, first-class answer, never a
  confident guess.

## Limits

- It will not certify that a claim is *true in the world* — it reports what the cited
  sources say and whether they are quotable.
- It will not assert anything its own code could not corroborate.
- If no model lane answers, it returns `503` and **does not bill**.
- Settlement happens before execution, so a paid call whose evidence does not check out
  returns a billed `unverifiable` verdict — never a fabrication.

## Discovery endpoints

| Path | Method | Returns |
|---|---|---|
| `/v1/verify/claims` | POST | 402 challenge → 200 verdicts |
| `/v1/verify/claims` | GET / HEAD | 402 challenge (discovery) |
| `/v1/extract` | POST | 402 challenge → 200 article |
| `/v1/extract` | GET / HEAD | 402 challenge (discovery) |
| `/v1/pricing` | GET | 200 price + rail metadata |
| `/.well-known/x402` | GET | 200 discovery document |
| `/x402.json` | GET | 200 discovery document |
| `/health` | GET | 200 liveness |

---

Verification is evidence reporting, not truth certification. Read every
`quote_verbatim` flag before trusting a verdict downstream.
