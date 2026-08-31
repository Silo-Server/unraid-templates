# Silo — Unraid Community Applications templates

Official [Unraid](https://unraid.net) Community Applications templates for
[Silo](https://github.com/Silo-Server/silo-server), a self-hosted media server for
films and series.

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

If you enable Unraid autostart, keep Redis and Silo-PostgreSQL ahead of Silo in Docker
startup order and configure a short wait after PostgreSQL. Silo exits when either dependency
is still unavailable; start it again once both services are ready.

Choose a unique `POSTGRES_PASSWORD` when installing Silo-PostgreSQL; no password is supplied
by the template. Use that same password in Silo's `DATABASE_URL`. If the password contains
reserved URL characters, percent-encode them in the connection string. `DATABASE_URL` and
`REDIS_URL` show as visible placeholders/examples in the Unraid form, not masked fields —
replace them with your own values before starting the container. Because `DATABASE_URL` is
unmasked, the database password appears in plaintext in the Unraid template/configuration.

### Networking note

Containers on Unraid's default `bridge` network **cannot resolve each other by name**.
The `DATABASE_URL` example and `REDIS_URL` default contain a literal `YOUR-UNRAID-IP`
placeholder that must be replaced with the server's LAN IP address. Substituting the
Compose-style hostnames (`postgres`, `redis`) will not work.

Users who prefer name resolution can create a custom Docker network and attach all three
containers to it.

### Media path

The template maps `/mnt/user/data` to the identical `/mnt/user/data` path inside Silo and
mounts it read-only. Keep films and series beneath that root, then add the relevant
subdirectories as libraries in Silo.

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

Hardware acceleration is optional; CPU-only hosts install without any GPU settings.

For Intel or AMD, if `/dev/dri` exists on the Unraid host, enable **Advanced View** and add
`--device=/dev/dri:/dev/dri` to **Extra Parameters**. Do not add an empty Device entry:
DockerMan renders that as the invalid argument `--device=''`. Device exposure has been
smoke-tested on an AMD Hawk Point iGPU; a configured media library is still required to
exercise an actual transcode.

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

## Manual template installation

Copy the XML onto the Unraid flash drive to make the templates available from the Docker tab:

```sh
cp templates/*.xml /boot/config/plugins/dockerMan/templates-user/
```

## Contributing

Read [CONTRIBUTING.md](CONTRIBUTING.md) before opening a pull request. Changes to
topology, required services, networking defaults, storage, or upgrade behavior
should start as an issue.

## License

The template XML and repository documentation are MIT licensed. The Silo name, logo, icons,
and other brand assets are © 2026 Silo Media L.L.C. and are not included in the MIT license;
see `NOTICE.txt` and `TRADEMARK.md`. Silo server source code is AGPL-3.0-or-later.
