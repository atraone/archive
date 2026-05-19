# Proof Archive

A cryptographically-sealed public record of web archives captured by [Proof](https://proof.atra.one).

Every entry in this repository was:
1. **Fetched** at a precise moment in time
2. **Hashed** with SHA-256 (content + location metadata)
3. **Signed** with Ed25519 by the Proof server
4. **Anchored** to the Bitcoin blockchain via [OpenTimestamps](https://opentimestamps.org)
5. **Timestamped** on Nostr — published to multiple independent relays instantly

---

## Repository structure

```
archives/
  YYYY-MM/
    arc_{id}/
      proof.json   ← cryptographic proof manifest
      README.md    ← readable archive export (Markdown)
```

Only archives explicitly marked **public** by their creator appear here.  
User identity is only included if the user has opted in to a public profile.

---

## Verifying a proof

### Quick (online)
Visit `https://proof.atra.one/verify/{archive-id}` and paste or upload the `proof.json`.

### Independent (offline)

**1. Verify the Ed25519 signature**

```bash
# Install the opentimestamps CLI
pip install opentimestamps-client

# Extract fields from proof.json
PAYLOAD=$(jq -r '.signatures.ed25519.payload' proof.json)
SIG=$(jq -r '.signatures.ed25519.signature' proof.json)
PUBKEY=$(jq -r '.signatures.ed25519.publicKey' proof.json)

# Verify (using openssl or any Ed25519 tool)
echo "$PAYLOAD" | xxd -r -p | openssl pkeyutl -verify -pubin \
  -sigfile <(echo "$SIG" | xxd -r -p) \
  -pkeyopt digest: -in /dev/stdin
```

**2. Verify the Bitcoin timestamp**

```bash
# Download the .ots receipt from R2 (link in proof.json) or this repo
ots verify arc_{id}.ots
```

This independently confirms the archive existed before a specific Bitcoin block — without trusting Proof's servers.

**3. Verify the Nostr event**

```bash
# Query any Nostr relay for the event ID in proof.json
EVENT_ID=$(jq -r '.anchors.nostr.eventId' proof.json)
curl "https://nostr.wine/e/$EVENT_ID"
```

---

## What Proof provides (and doesn't)

| Claim | Method | Independently verifiable? |
|---|---|---|
| Content was unmodified at capture time | SHA-256 hash | ✅ Yes |
| Capture was made by Proof servers | Ed25519 signature | ✅ Yes (with public key) |
| Capture existed before block N | Bitcoin OTS | ✅ Yes (offline) |
| Timestamp is approximately correct | Nostr `created_at` | ✅ Yes (multi-relay) |
| The archived URL actually resolved | HTTP status in proof.json | ⚠️ Trust Proof |

Proof does **not** claim legal admissibility in any jurisdiction. These records are strong technical evidence that a web resource existed and had specific content at a point in time.

---

## Public key

The Ed25519 public key used to sign all proofs is published at:
`https://api.proof.atra.one/api/public-key`

---

*Generated automatically. Do not edit manually.*
