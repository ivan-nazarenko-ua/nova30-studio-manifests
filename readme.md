# Nova30 — Publication Manifests

This repository contains automatically generated cryptographic manifests for posts published on [nova30.studio](https://nova30.studio).

## Purpose

Each manifest serves as a **proof-of-publication** — a timestamped, hash-based record that a particular piece of content existed at a given point in time. Git history provides an independent, tamper-evident log of when content was created, modified, published, or deleted.

## Structure

Manifests are stored as JSON files organized in a two-level directory tree:

```
<XX>/<YY>/post_<ID>.json
```

Directory levels are derived from the post ID (`ID % 29` and `ID / 29 % 29`).

## Manifest format (version `pm-1`)

```jsonc
{
  "manifest": {
    "manifestVersion": "pm-1",
    "normalizerVersion": "norm-v1",       // text normalization algorithm version
    "hashAlgorithm": "sha256",
    "contentRootHash": "abcd1234...",      // SHA-256 of the entire manifest (excluding this field)
    "prevContentRootHash": "ef567890...",   // links to the previous revision
    "manifestCreatedAt": "2026-02-26T..."
  },
  "author": {
    "publicId": "uuid-v4"                  // author's public identifier (no PII)
  },
  "post": {
    "id": 42,
    "canonicalUrl": "https://nova30.studio/post/42",
    "published": true,
    "deleted": false,
    "lastUpdatedAt": "2026-02-26T..."
  },
  "events": [                              // append-only lifecycle log
    { "type": "created",     "timestamp": "..." },
    { "type": "published",   "timestamp": "..." },
    { "type": "updated",     "timestamp": "..." }
  ],
  "content": {                             // per-locale content hashes
    "title":       { "uk": { "strictHash": "...", "wideHash": "..." }, "en": { ... } },
    "description": { "uk": { ... } },
    "body":        { "uk": { ... } }
  },
  "attachments": [                         // file hashes
    {
      "bucket": "image",
      "storagePath": "/2026/02/abc123.jpg",
      "mimeType": "image/jpeg",
      "bytes": 204800,
      "hashes": { "sha256": "..." }
    }
  ]
}
```

### Content hashes

Text content is normalized before hashing (Unicode NFKC, confusable character mapping, punctuation normalization, whitespace collapsing). Two hashes are computed per field per locale:

- **strictHash** — hash of the normalized text (exact match)
- **wideHash** — hash of the normalized text with punctuation stripped (fuzzy match)

### Integrity verification

`contentRootHash` is computed by sorting all keys recursively, serializing to JSON (without the `contentRootHash` field itself), and taking the SHA-256 of the result. This allows independent verification that no field has been modified after the manifest was generated.

## Publishing schedule

Manifests are committed and pushed daily at 03:30 server time (UTC).

## License

The manifest data in this repository is provided for transparency and proof-of-publication purposes. The content of the posts themselves is subject to copyright as described in the [Terms of Use](https://nova30.studio/en/texts/terms-of-use).
