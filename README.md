# Entropy digests

A copy, outside PrizeCore's own hosting, of the signed daily digests of the Entropy ledger (https://entropy.prizecore.io), the public record of
PrizeCore's competition draws and instant-win commitments. One file a UTC day, `digests/<year>/<day>.json`, written by
Entropy's nightly job once the day's digest exists and never replaced. Compare any day here with https://entropy.prizecore.io/digests and
https://entropy.prizecore.io/api/v1/digests/<day>: if the ledger were ever rewritten, the copy here would no longer match.

## A file

- `digestHash`: SHA-256 of the day's record hashes, in ledger order, joined by newlines (SHA-256 of nothing on a day with no records).
- `signature`: Ed25519 over the UTF-8 bytes of the 64-character `digestHash`, made with Entropy's signing key.
- `publicKey`: the raw 32-byte Ed25519 public key the signature checks against, as hex. It is also published at
  https://entropy.prizecore.io/digests and https://entropy.prizecore.io/.well-known/entropy.json (today: `3777f13ee338cfc18475ee0ebc29af45345e5c9f50d67f2fc0613d9f11516023`).
- `recordCount`, `firstRecordHash`, `lastRecordHash`: the day's size and ends, for a quick look.

## Checking a day

```js
// Node.js: recompute the digest from the ledger and check the signature against the copy kept here
const { createPublicKey, verify, createHash } = require('node:crypto');
const copy = require('./digests/2026/2026-10-21.json');
const live = await (await fetch('https://entropy.prizecore.io/api/v1/digests/' + copy.day)).json();
const recomputed = createHash('sha256').update(live.recordHashes.join('\n')).digest('hex');
const key = createPublicKey({ key: Buffer.concat([Buffer.from('302a300506032b6570032100', 'hex'), Buffer.from(copy.publicKey, 'hex')]), format: 'der', type: 'spki' });
console.log(recomputed === copy.digestHash, verify(null, Buffer.from(copy.digestHash, 'utf8'), key, Buffer.from(copy.signature, 'hex')));
```

Records hold ticket numbers, hashes, times and operators' own opaque ids: never a name, an email or an address.
