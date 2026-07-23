# Digital Passport for Artworks

URL:   https://ethical-tech-colab.github.io/digital-passport-artworks/                                                                                     

A browser-based prototype for issuing, verifying, and revoking digital passports for artworks — built entirely client-side with real cryptographic primitives.

> **Prototype notice:** Duplicate detection works within a single browser session only. Upload one image, issue its passport, then upload another image — the system compares them in real time. There is no persistent database; refreshing the page clears all memory. This demo shows the detection pipeline, not a production registry.

## What It Does

This single-page application walks through the full lifecycle of an artwork's digital passport:

1. **Upload** — submit artwork photos with an interactive crop tool that auto-detects the subject
2. **Fingerprint** — compute three independent fingerprints (SHA-256, perceptual dHash, and angle count)
3. **Similarity Search** — check against all previous submissions in the session using perceptual hashing, multi-scale keypoint matching (Harris corners + SIFT-style descriptors + RANSAC), optional MobileNet embeddings, and engineered feature vectors
4. **Forensic Signal Fusion** — run spectral/edge energy, noise-floor, block-relative ELA, and provenance marker checks
5. **Forgery-Risk Scoring & Routing** — compute a weighted risk score; auto-clear, flag for human review, or block based on similarity + risk
6. **Certificate Chain** — generate a real ECDSA P-256 trust chain (governance root → federated root → issuing CA) in-browser
7. **Passport Issuance** — the issuing CA signs the artwork record; duplicate submissions are blocked
8. **Revocation** — StatusList2021-style signed bitstring; instant revoke/reinstate
9. **Verification** — full chain-of-trust verification; edit the passport JSON to simulate tampering

## Key Features

- **Duplicate detection** — uploading the same artwork twice (even from a different angle) is caught by the similarity engine; the system blocks issuing a second passport and directs the user to retrieve the existing record
- **Human review gate** — inconclusive risk scores or unconfirmed similarity matches are routed to human review instead of auto-issuing
- **Real cryptography** — ECDSA P-256 key generation, signing, and verification via the Web Crypto API
- **Fully client-side** — nothing leaves the browser tab; no server, no database, no network calls (except the optional MobileNet model)

## How to Run

Open `index.html` in any modern browser. For full Web Crypto support, serve it over localhost:

```bash
# Python
python3 -m http.server 8080

# Node
npx serve .
```

Then visit `http://localhost:8080`.

## Tech Stack

- Vanilla HTML / CSS / JavaScript (single file, no build step)
- Web Crypto API (ECDSA P-256, SHA-256)
- Canvas API (dHash, color histograms, HOG descriptors, keypoint extraction, ELA)
- Optional: TensorFlow.js + MobileNet for real embedding-based similarity

## License

MIT

---

## Peer Review

The full independent academic peer review of this prototype is in [PEER-REVIEW.md](PEER-REVIEW.md) (also available as [Word](peer-review/digital-passport-artworks-Peer-Review.docx) under [`peer-review/`](peer-review/)).

**Recommendation:** Major revisions — for one routing defect and one attribution concern, not for the quality of the engineering, which is high. The in-app honesty about the forensic signals is held up in the review as the portfolio's example of good empirical practice.

**What the review found:**

- **A four-character substring bypasses the routing gate.** `hasProvenanceMarkers` decodes the first 200KB as latin1 and sets `hasC2pa` if the text contains `jumb`, `c2pa`, or `C2PA` anywhere — no JUMBF box parsing, no manifest signature verification. Appending those bytes to any file drops `metaRisk` from 0.65 to 0.10 and moves it from human review to auto-issue. The same happens by accident whenever those characters occur in a filename, comment, or binary run.
- **The forgery-risk score is 70% metadata presence, and that binary decides the outcome.** With weights ELA 0.10 / spectral 0.10 / noise 0.10 / provenance 0.70 and a binary `metaRisk`, files with EXIF or C2PA bytes score 7–37 and files without score 45–76 — non-overlapping ranges either side of the auto-issue gate at 35. No combination of the three forensic signals can move a file across it, so the signals shown with percentage bars and "+N pt to score" cannot change any routing outcome. EXIF is stripped by most platform pipelines and trivially forged, so the check runs in both wrong directions.
- **The demo mints credentials in the names of real institutions.** The chain is labelled "AABC × Bocconi / ICOM-UNESCO Council", "Italy Root — Carabinieri TPC", and "Museo Civico Archeologico", and the custodian field is locked to the issuing CA — so an exported passport asserts that a real art-crime authority stands behind an artwork record, from keys generated in the visitor's browser. Recommended fix: fictional placeholders plus a `"demo": true` field inside the signed payload, where it cannot be stripped without breaking the signature.
- **The self-signed root makes verification circular** and the verify step does not say so — the chain verifies against a root the same page load created.
- Minor: ELA is retained at 10% despite the UI correctly reporting it unreliable; the calibration sample is four photographs and one synthetic splice; the README presents the four forensic checks as peers; auto-reject is effectively unreachable (needs 76 of a possible 76).

**Reviewer's note on disclosure:** the substring bypass is written up here because the prototype is explicitly session-only with no persistent registry and nothing of value behind the gate. Fixing the check before this demo is linked more widely is the recommended order.
