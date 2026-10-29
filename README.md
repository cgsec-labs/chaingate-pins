# ChainGate advisory feed

This repository publishes the ChainGate advisory feed: npm package versions known to be malicious, one per line. A new
cut is published every hour and signed. It is made for the ChainGate runtime to read alongside its own signed seed, so
that a version listed here is blocked at install time. The file names say "pins", the runtime's word for a blocked
version.

Security notes and how to report a wrong entry: [SECURITY.md](SECURITY.md).

## Files

| URL | What it is |
|---|---|
| https://raw.githubusercontent.com/r-bedekar/chaingate-pins/main/pins/latest.jsonl | the current cut |
| https://raw.githubusercontent.com/r-bedekar/chaingate-pins/main/pins/latest.jsonl.sig | its signature |
| https://raw.githubusercontent.com/r-bedekar/chaingate-pins/main/pins/keys.json | the feed keys, delegated by the seed key |
| https://raw.githubusercontent.com/r-bedekar/chaingate-pins/main/pins/keys.json.sig | the seed key's signature over keys.json |

Each cut is also kept for 90 days as `pins/cuts/<cut time>.jsonl` with its `.sig`. The cut time is the header's `cut_at`
with the `-` and `:` removed, for example `20261029T110000Z`.

## Format

The first line is a header: `feed_version`, `cut_at`, `count`, `prev_cut_digest` and `prev_cut_at` (the previous cut,
so cuts form a chain), `pins_key_id`, and the advisory data it was made from (`label_input_digest`,
`label_input_metadata_cutoff`). Then there is one line per package version, sorted: `name`, `version`, `advisory_id`,
`also_advisories`, `record_id`, `source`, `evidence`, `added_at`. Every line is canonical JSON in ASCII.

## Verify a cut

The ChainGate seed key, the key built into the ChainGate runtime. Save it as `chaingate-seed-key.pem`:

```
-----BEGIN PUBLIC KEY-----
MCowBQYDK2VwAyEAP2W40LmbxTrDqDKaOpbfWD/xrbSPW4hz6RqQxZFte5E=
-----END PUBLIC KEY-----
```

1. `keys.json` is signed by the seed key, with Ed25519 over its bytes as they are. With OpenSSL 3:

   ```
   openssl pkeyutl -verify -pubin -inkey chaingate-seed-key.pem -rawin -in keys.json -sigfile keys.json.sig
   ```

2. Take the feed key from `keys.json`. It is the key with role `pins` whose `key_id` equals the cut header's
   `pins_key_id` and whose `valid_from` to `valid_to` window holds the cut's `cut_at`:

   ```
   K=$(head -1 latest.jsonl | jq -r .pins_key_id)
   { echo '-----BEGIN PUBLIC KEY-----'; jq -r --arg k "$K" '.keys[] | select(.key_id==$k and .role=="pins") | .spki_b64' keys.json; echo '-----END PUBLIC KEY-----'; } > pins-key.pem
   ```

3. The cut is signed by that key, with Ed25519 over the cut's SHA-256 written as 64 lower-case hex characters, with no
   newline:

   ```
   printf '%s' "$(sha256sum latest.jsonl | cut -d' ' -f1)" > msg.txt
   openssl pkeyutl -verify -pubin -inkey pins-key.pem -rawin -in msg.txt -sigfile latest.jsonl.sig
   ```

Each step should print `Signature Verified Successfully`. The ChainGate runtime makes the same checks, and also checks
that each cut links to the one it last accepted.
