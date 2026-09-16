# Decision Receipts

Safe utilities for consuming and verifying InterAI decision evidence without overstating what a receipt proves.

Decision Receipts helps a host distinguish a public receipt reference from owner-authenticated signed evidence, bind that evidence to an expected execution intent, and call InterAI's documented service-verification boundary without reserializing the signed payload.

```js
import {
  assertReceiptIntentDigest,
  verifyReceiptWithService,
} from "./index.mjs";

assertReceiptIntentDigest(ownerLookup, expectedExecutionIntentDigest);
await verifyReceiptWithService({ lookup: ownerLookup });
```

## Security model

The helper enforces several important rules:

- an anonymous `public_summary` is a reference only and is never accepted as authorization evidence;
- owner-authenticated lookup data must contain complete service-verifiable evidence;
- `verification.signed_payload` is treated as opaque and forwarded unchanged;
- the expected `execution_intent_digest` must match exactly when asserted;
- transport, HTTP, parse and invalid-signature outcomes fail closed.

Current InterAI receipt signatures use HMAC-SHA256 and are **service-verifiable by InterAI**. This repository does not provide or claim independent offline public-key verification.

## Why this exists

A durable decision receipt is useful only if consumers preserve its trust boundary. A public lookup link, a receipt ID, or a human-readable summary must not silently become permission to execute.

```text
receipt reference
      |
      +--> public summary -----------------> reference only
      |
      +--> owner-authenticated evidence
                    |
                    v
             exact intent match
                    |
                    v
          InterAI service verification
                    |
         valid? ----+---- no ---> fail closed
                    |
                   yes
                    v
             consumer continues
```

## API

- `requireOwnerSignedEvidence(lookup)` — rejects incomplete or public-summary-only evidence.
- `assertReceiptIntentDigest(lookup, expectedDigest)` — checks exact execution-intent binding.
- `verifyReceiptWithService(options)` — calls `POST /trust/verify-signature` and fails closed on verification uncertainty.
- `ReceiptVerificationError` — typed error with a machine-readable `code`.

`verifyReceiptWithService` defaults to `https://api.interailabs.dev` and a 5000 ms timeout.

## Non-goals

This repository is not a receipt issuer, signing implementation, key-management system, authorization engine or proof that a side effect occurred.

A DecisionReceipt is evidence of a decision. It is not itself proof that the action executed, and it is not a substitute for domain-specific controls.

## Run tests

Requires Node.js 20 or newer.

```bash
npm test
```

No runtime dependencies are required.

## Relationship to InterAI

This repository is an open-source evidence-consumption tool from InterAI Labs. [InterAI Risk Oracle](https://github.com/InterAILabs/ai-risk-oracle) is the related hosted pre-execution decision service.

This repository consumes documented public receipt fields and endpoints only. It contains no signing keys, private signing implementation, account data, Risk Oracle policy/scoring logic, trust intelligence, billing/storage internals or production operations.

Hosted Risk Oracle API: `https://api.interailabs.dev`

## License

Apache-2.0. See `LICENSE` and `NOTICE`.
