# Public keys for condado.dev

OpenPGP public keys for **condado.dev**, published via [Web Key Directory
(WKD)][wkd] so they can be discovered automatically from an email address — no
keyserver, no manual fingerprint exchange.

## Fetch a key automatically (WKD)

```bash
gpg --locate-external-keys gg@condado.dev
```

`gpg` derives the domain from the address, fetches the key from this site over
HTTPS, and imports it. Always confirm the fingerprint below before trusting it.

## Available keys

| Email            | Primary fingerprint                                 | Download                 |
| ---------------- | --------------------------------------------------- | ------------------------ |
| `gg@condado.dev` | `802A 939D ECC1 86E1 FD52  20CC 5FBF D625 E01E F53F` | [`gg.asc`](keys/gg.asc)  |

Direct armored download (handy for linking elsewhere):

```bash
curl -s https://openpgpkey.condado.dev/keys/gg.asc | gpg --import
```

## Verify

Check discovery and the served key with the online tool
<https://www.webkeydirectory.com/> (enter `gg@condado.dev`), or compare the
imported key's fingerprint against the table above.

---

Hosting, layout, and instructions for adding or rotating keys are in
[MAINTENANCE.md](MAINTENANCE.md).

[wkd]: https://datatracker.ietf.org/doc/draft-koch-openpgp-webkey-service/
