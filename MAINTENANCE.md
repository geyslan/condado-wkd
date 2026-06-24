# Maintenance - condado-wkd

Operational notes for the [Web Key Directory (WKD)][wkd] hosted at
`openpgpkey.condado.dev` (WKD *advanced method*). The public landing page lives
in [`README.md`](README.md); this file is the how-it-works / how-to-change-it
reference.

## Layout

```
_config.yml                                      # Jekyll: force-include .well-known
.well-known/
└── openpgpkey/
    └── condado.dev/
        ├── policy                              # required, intentionally empty
        └── hu/
            └── 6ctg9rxzxkd71npa1efp3rx3wcy4qcmm   # binary pubkey for gg@condado.dev
keys/
├── gg.asc                                      # armored copy, for human-facing links
└── aquasec.com/                                # link-only keys, grouped by owning domain
    └── geyslan.gregorio.asc                    # <localpart>.asc (not WKD-discoverable)
CNAME                                            # GitHub Pages custom domain
README.md                                        # rendered landing page (/)
MAINTENANCE.md                                   # this file
```

- Each file under `hu/` is the **binary** (`--no-armor`) OpenPGP public key for
  one email address. The filename is the [z-base-32][zb32] SHA-1 hash of the
  lowercased local part (the bit before `@`). These are what WKD clients fetch.
- `policy` must exist but is empty (it can carry WKD policy flags; we use none).
- `keys/*.asc` are **ASCII-armored** copies for humans - link them on a webpage,
  paste in a profile, etc. They are *not* part of WKD discovery; they are just
  static downloads (`curl …/keys/gg.asc | gpg --import`).

> **Why `_config.yml`:** GitHub Pages runs Jekyll, which by default excludes any
> file or folder starting with a dot. Without the `include: [".well-known"]`
> directive the entire WKD tree returns **404** while the rest of the site works
> - a silent breakage. Keep that file.

## Published keys

| Email            | Primary fingerprint                         | `hu/` filename                       |
| ---------------- | ------------------------------------------- | ------------------------------------ |
| gg@condado.dev   | `802A 939D ECC1 86E1 FD52  20CC 5FBF D625 E01E F53F` | `6ctg9rxzxkd71npa1efp3rx3wcy4qcmm` |

## Adding or updating a key

1. Find the WKD filename (hash) for the address:

   ```bash
   gpg --list-keys --with-wkd-hash <email>
   ```

   The line under the UID prints `<hash>@condado.dev` - `<hash>` is the filename.

2. Export the **binary** key for WKD:

   ```bash
   gpg --no-armor --export-options export-minimal \
       --export <email> > .well-known/openpgpkey/condado.dev/hu/<hash>
   ```

3. Export an **armored** copy for linking (optional but recommended):

   ```bash
   gpg --armor --export-options export-minimal --export <email> > keys/<name>.asc
   ```

4. Commit and push. GitHub Pages redeploys automatically.

One file per email address. Different people use different local parts (and so
different `hu/` files); multiple keys bound to the *same* address share one file
(export them all into it).

Because the binary `hu/` file and the armored `keys/*.asc` are two encodings of
the **same** key, regenerate both whenever you rotate or extend a key.

### Keys for domains you do not control

WKD is **domain-scoped**: a lookup for `user@example.com` is computed from and
fetched from `example.com`'s own WKD - never from condado.dev. So a key for an
address like `geyslan.gregorio@aquasec.com` **cannot** be made WKD-discoverable
from here; only the owner of `aquasec.com` can publish that. You may still host
it as an armored link-only download under `keys/<domain>/<localpart>.asc` (e.g.
`keys/aquasec.com/geyslan.gregorio.asc`) - a public key is public - it just
won't be found by `gpg --locate-keys`. Grouping by the owning domain keeps these
foreign keys clearly separated from condado.dev's own WKD-backed `keys/*.asc`.

## Hosting (GitHub Pages)

1. Push this repo; Settings -> Pages -> Source: `main` / root.
2. Custom domain: `openpgpkey.condado.dev` (matches the `CNAME` file), enable
   **Enforce HTTPS** (only available once the Let's Encrypt cert has issued -
   may lag the DNS check by minutes to ~1h).
3. At the DNS provider (Namecheap), add:

   ```
   Type: CNAME   Host: openpgpkey   Value: <github-user>.github.io.
   ```

   This only adds the `openpgpkey` subdomain - existing MX / email-forwarding
   records on the apex are untouched.

> **Note - browser mail clients:** browser-based clients (e.g. Proton webmail)
> need `Access-Control-Allow-Origin: *` to discover WKD keys; `gpg` does not.
> GitHub Pages already sends this header (via its Fastly CDN), so no extra config
> is required - verify with:
>
> ```bash
> curl -sI https://openpgpkey.condado.dev/.well-known/openpgpkey/condado.dev/hu/6ctg9rxzxkd71npa1efp3rx3wcy4qcmm \
>   | grep -i access-control-allow-origin   # -> access-control-allow-origin: *
> ```

## Methods: advanced only (by design)

WKD has two layouts (see [the validator](https://www.webkeydirectory.com/)):

- **Advanced** (what we use): served from the dedicated `openpgpkey.condado.dev`
  subdomain at `.../openpgpkey/condado.dev/hu/<hash>`.
- **Direct**: served from the apex at `https://condado.dev/.well-known/openpgpkey/hu/<hash>`.

Only **advanced** is configured, and that is sufficient: every WKD client tries
advanced first and falls back to direct only if advanced is absent. Direct is
also impractical here - the apex has no web host (just DNS + email forwarding),
and a GitHub Pages repo allows a single custom domain (already `openpgpkey.`),
so serving the apex would need a separate host. If a real website ever lives on
`condado.dev`, adding the apex `.well-known/openpgpkey/` tree would then give
direct for free - but there is no WKD reason to do it.

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
