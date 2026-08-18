# Silo — Unraid Community Applications templates

Official [Unraid](https://unraid.net) Community Applications templates for
[Silo](https://github.com/Silo-Server/silo-server), a self-hosted media streaming server
for movies, shows, music and books.

## Templates

| Template | Image | Purpose |
| --- | --- | --- |
| `templates/silo.xml` | `ghcr.io/silo-server/silo-server` | The media server |
| `templates/silo-postgres.xml` | `pgvector/pgvector:pg18` | PostgreSQL 18 + pgvector |

Unraid's Community Applications installs one container per template — there is no Compose
support. Silo's PostgreSQL dependency ships as a companion template because it requires
PostgreSQL 18 with the `pgvector` extension. Redis does not require a Silo-specific image;
install any Redis 6.2-or-newer container from Community Applications, or use an existing
Redis service.

## Install order

1. **Silo-PostgreSQL** — start it, confirm it is healthy
2. **Redis 6.2+** — install a generic Redis container or use an existing Redis service
3. **Silo** — set `SECRET_KEY`, `DATABASE_URL` and `REDIS_URL`, then start

Redis is required. Silo will not start until it can reach both PostgreSQL and Redis. A
dedicated Redis instance is recommended, particularly if you run more than one Silo server.

### Networking note

Containers on Unraid's default `bridge` network **cannot resolve each other by name**.
The `DATABASE_URL` and `REDIS_URL` defaults contain a literal `YOUR-UNRAID-IP` placeholder
that must be replaced with the server's LAN IP address. Substituting the Compose-style
hostnames (`postgres`, `redis`) will not work.

Users who prefer name resolution can create a custom Docker network and attach all three
containers to it.

### SECRET_KEY

Silo refuses to start without `SECRET_KEY` (see `internal/config/bootstrap.go`). Generate one
from an Unraid terminal:

```sh
openssl rand -base64 48
```

Back it up separately from database dumps — losing it makes encrypted secrets unrecoverable.

### Hardware transcoding

The Silo template maps `/dev/dri` for Intel and AMD integrated GPUs. Remove that entry on
servers without an iGPU. Verified working on AMD Radeon 780M (`amdgpu`, `renderD128`).

## Before submitting to Community Applications

These items must be complete before submission:

- [x] **Support links** — all templates link to Silo's GitHub Issues page.
      Community Applications accepts forums, issue trackers, or project-help pages
      for `<Support>`; a dedicated Unraid forum thread is optional
- [ ] **Make this repo public.** It is currently private. Community Applications fetches
      templates *and* icons over `raw.githubusercontent.com`, so a private repo will fail
      Validate/Scan and show broken icons
- [x] **Icons** — synchronized from the official `Silo-Server/silo-branding` repository
      at `6278e96`.
      CA listing icons use `logo/png/silo-project-icon-256.png`; `icons/silo-1024.png`
      preserves the corresponding 1024 px render. `silo-postgres.png` intentionally uses
      the same unmodified project icon so the official template suite reads as one family.
- [ ] **Confirm 2FA** is enabled on the maintainer's GitHub account and acknowledge it
      during submission, as required by CA policy
- [ ] Run **Validate** then **Scan** at <https://ca.unraid.net/submit>

### Known gaps versus Unraid convention

- **No `PUID`/`PGID` support.** The Silo image has no `USER` directive and no
  `gosu`/`su-exec` step, so it runs as root. This is accepted on CA but is not the norm —
  most media containers let users run as `99:100` (`nobody:users`). Worth addressing upstream.

## Testing locally before submission

Templates can be trialled without submitting anything. Copy the XML onto the Unraid flash
drive and it appears in the Docker tab as though installed from CA:

```sh
cp templates/*.xml /boot/config/plugins/dockerMan/templates-user/
```

## License

The templates in this repository are MIT licensed. Silo itself is AGPL-3.0.
