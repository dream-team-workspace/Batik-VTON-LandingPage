# Deployment

The CoBatik landing page is deployed by GitHub Actions
(`.github/workflows/ci-cd.yml`):

- **Pull requests** — the `lint` job validates the HTML pages
  (`index.html`, `en/index.html`) with
  [html-validate](https://html-validate.org/).
- **Push to `master`** — the `deploy` job rsyncs the site (`index.html`,
  `en/index.html`, `robots.txt`, `sitemap.xml`, `assets/`) to the server and
  runs a smoke test against <https://cobatik.dream-team.works/> and
  <https://cobatik.dream-team.works/en/>.

## How it works

- The files are synced as the `deploy` user over key-only SSH (direct IPv4
  connection to `139.99.88.23`; no Cloudflare tunnel involved in the deploy
  step).
- Webroot: `/var/www/cobatik.dream-team.works` (owned by `deploy:deploy`).
- nginx serves the webroot on `127.0.0.1:8080` (not port 80), and the
  server's Cloudflare Tunnel (`ovh-server`) routes
  `cobatik.dream-team.works` to `http://localhost:8080`.

## Site structure

- `/` — Bahasa Indonesia (default language, referenced by
  `hreflang="x-default"`).
- `/en/` — English.

Both pages cross-reference each other with `rel="alternate"` hreflang tags,
carry their own canonical URL, Open Graph/Twitter metadata and JSON-LD
(`WebSite` + `MobileApplication`). `robots.txt` and `sitemap.xml` are
deployed at the site root.

## Required repository secrets

| Secret           | Value                                                                    |
| ---------------- | ------------------------------------------------------------------------ |
| `DEPLOY_SSH_KEY` | Private key of the `github-actions-cobatik` deploy key (Ed25519, no passphrase). |

The matching public key is installed in `deploy@`'s
`~/.ssh/authorized_keys` on the server. To rotate it, generate a new
keypair, append the public key on the server, and update this secret.
