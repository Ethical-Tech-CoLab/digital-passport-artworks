# Digital Passport for Artworks

A browser-based prototype for issuing, verifying, and revoking digital passports for artworks — built entirely client-side with real cryptographic primitives.

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
