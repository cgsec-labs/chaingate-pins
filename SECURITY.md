# Security

This repository publishes the ChainGate advisory feed: npm package versions known to be malicious, one per line. A new
cut is published every hour and signed.

- **Verify before use.** The signature in `latest.jsonl.sig` is Ed25519 over the SHA-256 hex of `latest.jsonl`. It is made
  by the key named in the cut's header, which `keys.json` delegates. `keys.json` is signed by the ChainGate seed key. The
  README gives the commands.
- **A stolen feed key.** Someone holding a stolen feed key could publish a cut that adds wrong entries or leaves out
  entries. The versions in ChainGate's own signed seed are not affected. Every cut is kept for 90 days, so a change can be
  seen and traced. A feed key is valid for at most 90 days and is replaced before it expires.
- **What GitHub sees.** GitHub serves these files and sees the address of every client that fetches them, as it does for
  the seed download.
- **Rate limits.** raw.githubusercontent.com limits requests that are not signed in. An hourly fetch per client is far
  below that limit.
- **History.** This branch is never force-pushed or deleted. Each cut is an ordinary commit.

Report a wrong entry or a security problem to the address in the ChainGate repository's
[SECURITY.md](https://github.com/r-bedekar/chaingate/blob/main/SECURITY.md).
