# sentinelx-app-web

Public web presence for [SentinelX Cloud](https://sentinelx.app),
the open-source MCP server by [Pensa Software](https://pensa.ar).

Served from `/var/www/get.sentinelx.app/` on the production host
(`orion`). nginx routes the following domains to this content:

- `https://sentinelx.app/` — primary landing
- `https://www.sentinelx.app/` — www redirect
- `https://get.sentinelx.app/about` — alias for the landing
- `https://get.sentinelx.app/` — the one-line installer endpoint
  (returns `install.sh`, see notes below)

## Layout

```
.
├── index.html          The landing page
├── privacy.html        Privacy policy
├── terms.html          Terms of service
├── favicon.svg         Master logo asset — used as favicon AND served
│                       as /logo.svg via nginx alias (one source of truth)
├── favicon.ico         Legacy favicon for older browsers
├── favicon.png         PNG fallback for clients that prefer it
├── logo.png            Optional larger PNG version of the logo
└── .gitignore
```

## What's NOT in this repo

Things that live in `/var/www/get.sentinelx.app/` but are **deliberately
not tracked**:

### `install.sh` and `enroll.py`

These are the **agent installer scripts**. The canonical source of
truth lives in:

  https://github.com/pensados/sentinelx-cloud-installer

They get copied into `/var/www/get.sentinelx.app/` at deploy time so
that `https://get.sentinelx.app/install.sh` resolves to a fresh
version. To update them:

```bash
# On orion, after pulling sentinelx-cloud-installer:
sudo cp /home/carlos/projects/sentinelx-cloud/sentinelx-cloud-installer/install.sh \
        /var/www/get.sentinelx.app/install.sh
sudo cp /home/carlos/projects/sentinelx-cloud/sentinelx-cloud-installer/enroll.py \
        /var/www/get.sentinelx.app/enroll.py
sudo chown www-data:www-data /var/www/get.sentinelx.app/install.sh \
                              /var/www/get.sentinelx.app/enroll.py
```

(There's a TODO to automate this with a post-merge git hook, but for
now it's a manual two-line copy after each installer release.)

### `demo.mp4`

The product demo video is ~350 MB — too big for git. It lives in
`/var/www/get.sentinelx.app/demo.mp4` and is served directly by nginx.
If the server is reinstalled from scratch, restore it from backups.

Eventually it'll move to a CDN (Cloudflare R2 / Backblaze B2) and
be referenced as `https://media.sentinelx.app/demo.mp4` from `index.html`.

## Deploy workflow

Files in this repo ARE the live website — the working tree IS
`/var/www/get.sentinelx.app/`. There is no separate "deploy" step:
editing a file here changes what nginx serves immediately.

To pull changes onto the running server:

```bash
ssh orion
cd /var/www/get.sentinelx.app
git pull
```

nginx serves the new content on the next request. No restart needed.

Caveat: files owned by `www-data` may require sudo or matching group
membership to edit. The `+` after the perms (e.g. `drwxrwsr-x+`)
indicates an ACL that grants editor access to specific users.

## License

Apache License 2.0 — see [LICENSE](./LICENSE).

Copyright 2026 Carlos Javier Torres Pensa
Pensa Software® — https://sentinelx.app
