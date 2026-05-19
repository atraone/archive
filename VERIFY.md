# Independent Verification Guide

This guide explains how to verify a Proof archive **without trusting Proof's servers**.

## What you need

- A `proof.json` file (from this repo or from `https://api.proof.atra.one/api/proof/{id}`)
- For Bitcoin verification: the `.ots` receipt file (stored in this repo alongside `proof.json`)
- Standard Unix tools: `openssl`, `python3`, `jq`
- Optionally: `opentimestamps-client` Python package

---

## Step 1 — Verify the content hash

The `content.sha256` field is a SHA-256 hash of the raw HTML at capture time.

```bash
# If you have the original HTML:
sha256sum captured_page.html
# Compare to: .content.sha256 in proof.json
```

## Step 2 — Verify the Ed25519 signature

```python
import json, hashlib
from cryptography.hazmat.primitives.asymmetric.ed25519 import Ed25519PublicKey
from cryptography.hazmat.primitives.serialization import load_der_public_key

proof = json.load(open('proof.json'))

# Reconstruct the signed payload
payload = proof['signatures']['ed25519']['payload']  # "contentHash:locationHash:capturedAt"

# Load the public key (SPKI DER hex)
pub_bytes = bytes.fromhex(proof['signatures']['ed25519']['publicKey'])
pub_key = load_der_public_key(pub_bytes)

# Verify
sig_bytes = bytes.fromhex(proof['signatures']['ed25519']['signature'])
pub_key.verify(sig_bytes, payload.encode())
print("✅ Signature valid")
```

## Step 3 — Verify the Bitcoin timestamp

```bash
pip install opentimestamps-client

# The .ots file is stored alongside proof.json in this repo
ots verify arc_{id}_alice.ots

# Expected output:
# Success! Bitcoin block {N} attests data existed as of {date}
```

## Step 4 — Verify the Nostr timestamp

```python
import json, websocket

proof = json.load(open('proof.json'))
event_id = proof['anchors']['nostr']['eventId']

# Query any public Nostr relay
ws = websocket.create_connection("wss://relay.damus.io")
ws.send(json.dumps(["REQ", "1", {"ids": [event_id]}]))
msg = json.loads(ws.recv())
print(f"Event created_at: {msg[2]['created_at']}")
ws.close()
```

---

## Trust model

These verification steps are fully independent — they use Bitcoin's blockchain and the
Nostr network, neither of which is controlled by Proof. Even if Proof's servers are
compromised or shut down, existing proofs remain verifiable forever.
