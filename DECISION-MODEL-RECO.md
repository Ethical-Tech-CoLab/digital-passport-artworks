# Decision-model recommendation: local-first duplicate review

Date: 2026-10-11. Status: offline research proposal.
Scope: this browser prototype only; public-source analysis.

## Architecture constraint

Reviewed [README](https://github.com/Ethical-Tech-CoLab/digital-passport-artworks/blob/cdda2bb64ed30f625d0d2778b4b1f31aacf24805/README.md),
revision `cdda2bb64ed30f625d0d2778b4b1f31aacf24805`.
This assessment is documentation-led, not a complete implementation review.

The prototype runs client-side, with session-scoped duplicate memory, image
hash/keypoint evidence, optional MobileNet features, forensics, real ECDSA
signatures and human-review routing. Resetting the session clears memory.

**Recommendation: do not add a cloud decision API by default.** It would change
the no-server/data-egress contract and cannot be secured by embedding a key
in browser code.

## Useful bounded investigation

Build an offline evaluation set of ambiguous duplicate candidates. Preserve
the original image/hash evidence and inspect whether a local metadata-pair
classifier adds useful review information:

- same physical object;
- related work, edition or reproduction;
- different object;
- insufficient evidence.

Treat metadata agreement as a separate signal, never as proof that two images
show the same original. Exact image and cryptographic checks stay in code.

## Model choices

Start with the existing image pipeline, fuzzy metadata rules and a licensed
local classifier/cross-encoder. Measure its browser or local-server memory,
latency and loading footprint before proposing deployment.

Jev's [entity-alignment recipe](https://docs.typesafe.ai/cookbooks/entity_alignment.md)
is an architectural reference, not a locally downloadable model. Jev,
Microsoft-Decision-1 and Sage require their hosted interfaces for the researched
exact models. A cloud comparison is permissible only in an independently
approved opt-in architecture using non-sensitive fixtures and protected keys.

## Evaluation and constraints

Group train/test by physical object and capture session. Include reproductions,
same artist/title, crops, altered lighting, conflicting dimensions, missing
metadata and new sessions with no prior records.

Measure candidate recall, false-duplicate proposals, accepted-match precision,
calibration, reviewer effort, memory and latency. Check pair-order symmetry,
option order and uncertainty. Test the reset/privacy behavior explicitly.

Initial acceptance: existing signature verification, review routing and
session-reset behavior remain unchanged. No automatic merge, issuance or
authenticity claim. Missing metrics or privacy approval block rollout.

## Next step

Keep this as an offline benchmark until there is measured added value beyond
the deterministic pipeline. If cloud access is later proposed, document data
consent, server-side secrets, retention and failure behavior in a separate
architecture change. No runtime change is requested by this recommendation.
