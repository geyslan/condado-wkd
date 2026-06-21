# condado-wkd

Static [Web Key Directory (WKD)][wkd] for **condado.dev**, served over the
`openpgpkey.condado.dev` subdomain (WKD *advanced method*).

It lets anyone fetch a verified public key straight from the domain:

```bash
gpg --locate-external-keys gg@condado.dev
# or force the WKD method explicitly:
gpg --auto-key-locate clear,wkd --locate-keys gg@condado.dev
```

No keyserver, no manual fingerprint exchange — the key is bound to the email
address and hosted under the domain we control.

## Layout

```
.well-known/
└── openpgpkey/
    └── condado.dev/
        ├── policy                              # required, intentionally empty
        └── hu/
            └── 6ctg9rxzxkd71npa1efp3rx3wcy4qcmm   # binary pubkey for gg@condado.dev
keys/
└── gg.asc                                      # armored copy, for human-facing links
CNAME                                            # GitHub Pages custom domain
```

- Each file under `hu/` is the **binary** (`--no-armor`) OpenPGP public key for
  one email address. The filename is the [z-base-32][zb32] SHA-1 hash of the
  lowercased local part (the bit before `@`). These are what WKD clients fetch.
- `policy` must exist but is empty (it can carry WKD policy flags; we use none).
- `keys/*.asc` are **ASCII-armored** copies for humans — link them on a webpage,
  paste in a profile, etc. They are *not* part of WKD discovery; they are just
  static downloads (`curl …/keys/gg.asc | gpg --import`).

## Published keys

| Email            | Primary fingerprint                         | `hu/` filename                       |
| ---------------- | ------------------------------------------- | ------------------------------------ |
| gg@condado.dev   | `802A 939D ECC1 86E1 FD52  20CC 5FBF D625 E01E F53F` | `6ctg9rxzxkd71npa1efp3rx3wcy4qcmm` |

## Adding or updating a key

1. Find the WKD filename (hash) for the address:

   ```bash
   gpg --list-keys --with-wkd-hash <email>
   ```

   The line under the UID prints `<hash>@condado.dev` — `<hash>` is the filename.

2. Export the key in WKD format (binary, minimal):

   ```bash
   gpg --no-armor --export-options export-minimal \
       --export <email> > .well-known/openpgpkey/condado.dev/hu/<hash>
   ```

3. Commit and push. GitHub Pages redeploys automatically.

One file per email address. Different people use different local parts (and so
different `hu/` files); multiple keys bound to the *same* address share one file
(export them all into it).

To also publish an armored copy for linking, export the same key with `--armor`:

```bash
gpg --armor --export-options export-minimal --export <email> > keys/<name>.asc
```

### Keys for domains you do not control

WKD is **domain-scoped**: a lookup for `user@example.com` is computed from and
fetched from `example.com`'s own WKD — never from condado.dev. So a key for an
address like `geyslan.gregorio@aquasec.com` **cannot** be made WKD-discoverable
from here; only the owner of `aquasec.com` can publish that. You may still host
it as an armored link-only download under `keys/` (a public key is public) — it
just won't be found by `gpg --locate-keys`.

## Hosting (GitHub Pages)

1. Push this repo; Settings → Pages → Source: `main` / root.
2. Custom domain: `openpgpkey.condado.dev` (matches the `CNAME` file), enable
   **Enforce HTTPS**.
3. At the DNS provider (Namecheap), add:

   ```
   Type: CNAME   Host: openpgpkey   Value: <github-user>.github.io.
   ```

   This only adds the `openpgpkey` subdomain — existing MX / email-forwarding
   records on the apex are untouched.

> **Note — browser mail clients:** GitHub Pages cannot set CORS headers. `gpg`
> does not need them, but browser-based clients (e.g. Proton webmail) require
> `Access-Control-Allow-Origin: *` to discover WKD keys. If you need that, host
> on Cloudflare Pages and add a `_headers` file:
>
> ```
> /.well-known/openpgpkey/*
>   Access-Control-Allow-Origin: *
> ```

## Verify

```bash
# end-to-end, once DNS + Pages are live:
gpg --locate-external-keys gg@condado.dev

# raw fetch of the hu file:
curl -s https://openpgpkey.condado.dev/.well-known/openpgpkey/condado.dev/hu/6ctg9rxzxkd71npa1efp3rx3wcy4qcmm | gpg --import

# armored copy (human-facing link):
curl -s https://openpgpkey.condado.dev/keys/gg.asc | gpg --import

# online checker:
#   https://www.webkeydirectory.com/  (enter gg@condado.dev)
```

[wkd]: https://datatracker.ietf.org/doc/draft-koch-openpgp-webkey-service/
[zb32]: https://www.rfc-editor.org/rfc/rfc6189
