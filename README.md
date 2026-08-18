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

Choose a unique `POSTGRES_PASSWORD` when installing Silo-PostgreSQL; no password is supplied
by the template. Use that same password in Silo's `DATABASE_URL`. If the password contains
reserved URL characters, percent-encode them in the connection string. Connection URLs are
masked in the Unraid form because they may contain credentials.

### Networking note

Containers on Unraid's default `bridge` network **cannot resolve each other by name**.
The `DATABASE_URL` example and `REDIS_URL` default contain a literal `YOUR-UNRAID-IP`
placeholder that must be replaced with the server's LAN IP address. Substituting the
Compose-style hostnames (`postgres`, `redis`) will not work.

Users who prefer name resolution can create a custom Docker network and attach all three
containers to it.

### Media path

The template maps `/mnt/user/data` to the identical `/mnt/user/data` path inside Silo and
mounts it read-only. Keep movies, shows, music, books and audiobooks beneath that root, then
add the relevant subdirectories as libraries in Silo. There is no separate Books mapping.

### SECRET_KEY

Silo refuses to start without `SECRET_KEY` (see `internal/config/bootstrap.go`). Generate one
from an Unraid terminal:

```sh
openssl rand -base64 48
```

Back it up separately from database dumps — losing it makes encrypted secrets unrecoverable.

### Persistent data

The Silo container uses one appdata mapping: `/mnt/user/appdata/silo` on the host to
`/var/lib/silo` in the container. It covers installed plugins, compatibility assets and
other local Silo state. Back it up together with the separate Silo-PostgreSQL data directory
and your `SECRET_KEY`.

### Hardware transcoding

Hardware acceleration is optional; CPU-only hosts install without either GPU field configured.

For Intel or AMD, if `/dev/dri` exists on the Unraid host, set the advanced
**Intel / AMD GPU Device** field to `/dev/dri`. Verified working on AMD Radeon 780M
(`amdgpu`, `renderD128`).

For NVIDIA NVENC/NVDEC:

1. Install the **Nvidia-Driver** plugin from Community Applications and reboot if prompted.
2. Open the plugin settings and copy the desired GPU UUID.
3. Edit the Silo container with **Advanced View** enabled and add `--runtime=nvidia` to
   **Extra Parameters**.
4. Enter the UUID in **NVIDIA GPU UUID** (or use `all`). Leave
   `NVIDIA_DRIVER_CAPABILITIES` set to `compute,video,utility`.

Silo's default `auto` hardware-acceleration mode detects the exposed NVIDIA device and selects
NVENC. The template does not force the NVIDIA runtime because doing so would prevent Silo from
starting on hosts without the plugin/runtime installed.

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

The template XML and repository documentation are MIT licensed. The Silo name, logo, icons,
and other brand assets are © 2026 Silo Media L.L.C. and are not included in the MIT license;
see `NOTICE.txt` and `TRADEMARK.md`. Silo server source code is AGPL-3.0-or-later.
