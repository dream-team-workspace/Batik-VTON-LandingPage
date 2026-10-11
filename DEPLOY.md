# Deployment

The CoBatik landing page is deployed by GitHub Actions
(`.github/workflows/ci-cd.yml`):

- **Pull requests** — the `lint` job validates `index.html` with
  [html-validate](https://html-validate.org/).
- **Push to `master`** — the `deploy` job rsyncs `index.html` and `assets/`
  to the server and runs a smoke test against
  <https://cobatik.dream-team.works/>.

## How it works

- The files are synced as the `deploy` user over key-only SSH (direct IPv4
  connection to `139.99.88.23`; no Cloudflare tunnel involved in the deploy
  step).
- Webroot: `/var/www/cobatik.dream-team.works` (owned by `deploy:deploy`).
- nginx serves the webroot on `127.0.0.1:8080` (not port 80), and the
  server's Cloudflare Tunnel (`ovh-server`) routes
  `cobatik.dream-team.works` to `http://localhost:8080`.

## Required repository secrets

| Secret           | Value                                                                    |
| ---------------- | ------------------------------------------------------------------------ |
| `DEPLOY_SSH_KEY` | Private key of the `github-actions-cobatik` deploy key (Ed25519, no passphrase). |

The matching public key is installed in `deploy@`'s
`~/.ssh/authorized_keys` on the server. To rotate it, generate a new
keypair, append the public key on the server, and update this secret.
