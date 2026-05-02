# ular-maldives-data

Distribution-only repository for the [`ular-maldives`](https://maldives.ular.io) desktop app. The repo holds two files and nothing else:

| File | Purpose |
|---|---|
| `manifest.json` | Plain-text. Version, size, sha256, AES-GCM iv/authTag, HMAC-SHA256 signature. The app fetches this on every launch to decide whether an update is available. |
| `inventory.bin` | AES-256-GCM ciphertext of the canonical inventory YAML (resorts, villas, meal plans). The app decrypts in memory and UPSERTs into its local SQLite. |

Updates land here a few times a year as the curated inventory is refreshed.

## Why is this public?

The app's main repo is private, but its *bundled* and *runtime-fetched* data needs to be reachable without authentication on every install. Hosting the data here as a separate public repo is the simplest way to do that — no auth, no CDN, no rate-limit issues.

## Why is the data encrypted?

The encryption raises the bar against casual scraping (a bot would have to reverse-engineer the app to extract the key). It is **not** strong protection against a determined attacker — symmetric keys live in the client, which is an inherent limitation of any client-side decryption scheme. The data is hotel inventory anyway, not a secret.

## How is this built?

In the main app repo:

```
scripts/inventory/
├── inventory.yaml      # canonical source, edited by the maintainer
├── guides/<slug>.md    # per-resort curation guide for AI agents
└── build.mjs           # validate → AES-GCM encrypt → manifest sign

→ pnpm inventory:build  # writes both data/<this repo> and resources/<bundled>
```

See `docs/inventory-distribution.md` in the main repo for the full schema, build pipeline, and app-side consumption flow.

## Don't bother trying to decrypt

The key is not stored in this repo and is not derivable from anything here. Decryption requires the matching app binary or the maintainer's private `.keys/` files.
