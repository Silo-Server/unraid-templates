# Silo — Unraid Community Applications templates

Official [Unraid](https://unraid.net) Community Applications templates for
[Silo](https://github.com/Silo-Server/silo-server), a self-hosted media server for
films and series.

## Templates

| Template | Image | Purpose |
| --- | --- | --- |
| `templates/silo.xml` | `ghcr.io/silo-server/silo-server:aio` | Silo with bundled PostgreSQL and Redis, and optional Meilisearch |
| `templates/silo-postgres.xml` | `pgvector/pgvector:pg18` | Optional external PostgreSQL for installations that manage the database separately |

The Silo AIO image runs the server and its bundled services in one container.
Separate PostgreSQL and Redis containers are no longer required. PostgreSQL
handles search by default; bundled Meilisearch is off.

## New installation

1. Install **Silo** from Community Applications.
2. Generate a `SECRET_KEY` in an Unraid terminal with `openssl rand -base64 48`
   and enter it in the template.
3. Choose your media and appdata paths. Use an empty appdata directory for a
   fresh installation; the default is `/mnt/user/appdata/silo-aio`.
4. Start the container and open its WebUI to create the administrator account
   and configure libraries.

The WebUI uses host port `8090`. Optional Jellyfin-compatible and
Audiobookshelf-compatible listeners use `8096` and `13378`; publishing those
ports does not enable the features in Silo. Change a host port if another
container already uses it.

The template sets `--restart=on-failure --stop-timeout=120 --memory=4g` in
**Extra Parameters**. These restart a failed container, allow two minutes for
Silo and PostgreSQL to shut down, and limit container memory to 4 GiB. Adjust
the memory limit for your workload. Preserve the restart and shutdown options
when adding GPU parameters. Unraid autostart is a separate setting; enable it
if Silo should start with the array. No companion-container startup order is
needed for the default AIO installation.

### Bundled and external services

These settings are available in **Advanced View**:

| Setting | Default | Behavior |
| --- | --- | --- |
| `DATABASE_URL` | Empty | Uses bundled PostgreSQL. Set an external PostgreSQL 18 connection URL with pgvector to disable the bundled database. |
| `ENABLE_REDIS` | `true` | Starts bundled Redis. Set `false` to run without Redis when no enabled feature needs it. |
| `REDIS_URL` | Empty | Set an external Redis 6.2-or-newer URL to disable bundled Redis. Overrides `ENABLE_REDIS`. |
| `ENABLE_MEILISEARCH` | `false` | Set `true` to start bundled Meilisearch and configure it as the search provider. |

Silo needs a PostgreSQL database, so there is no database-off mode without an
external `DATABASE_URL`. The template masks both connection URLs because they
can contain passwords; their saved values still need to be protected with the
Unraid configuration.

Meilisearch requires Redis for paged searches. Enabling bundled Meilisearch,
or setting `SILO_SEARCH_PROVIDER=meilisearch` for an external provider, starts
bundled Redis even when `ENABLE_REDIS=false`, unless `REDIS_URL` points elsewhere.
To run without Redis, leave `REDIS_URL` empty, set `ENABLE_REDIS=false`, leave Meilisearch off,
and use PostgreSQL search. Watch Party requires Redis.

Setting `ENABLE_MEILISEARCH=false` stops the bundled service. If the image
configured that search provider, it switches search back to PostgreSQL. It
does not clear a separately configured external Meilisearch provider; change
that provider in Silo's search settings if you no longer want to use it.

Bundled PostgreSQL and Redis are private to the container by default. No host
ports for either are published. When using external services on Unraid's
standard `bridge` network, use a reachable host address in the connection URL;
containers on that network cannot resolve each other by name. A user-defined
Docker network can provide container-name resolution.

### Media and permissions

The template maps `/mnt/user/data` to the same path inside Silo, read-only.
Choose your media share and add its film and series folders as libraries.
Silo does not modify the media files.

`PUID=99` and `PGID=100` use Unraid's `nobody` user and `users` group for Silo.
Change them if another user or group owns your media, and make sure that
identity can read the files and traverse their parent directories. PostgreSQL,
Redis, and Meilisearch keep their own service accounts inside the container.

### SECRET_KEY and persistent data

`SECRET_KEY` encrypts stored credentials. Back it up separately from the data;
losing it makes encrypted secrets unrecoverable. Keep the same key when
restoring an installation.

The **Appdata** mapping mounts `/mnt/user/appdata/silo-aio` at `/config`.
It contains Silo state, plugins, artwork, and the bundled services' data.
Stop the container before making a file backup of this directory so the
bundled PostgreSQL database is shut down cleanly. External databases need
separate backups.

**Transcode Scratch** defaults to `/mnt/user/appdata/silo-transcode` on the
host, mapped to `/tmp/silo-transcode`. Choose fast local storage with enough
space for active streams. A host path of `/tmp/silo-transcode` uses Unraid's
RAM-backed temporary storage instead; only use it if enough memory is
available. This scratch data does not need backing up.

The **Download Artifacts** mapping (Advanced View) holds transcoded copies that Silo
prepares for offline downloads when an administrator enables transcoded downloads. It
defaults to `/mnt/user/appdata/silo-download-artifacts`. The files can be large and Silo
recreates them on demand, so the folder can be left out of backups. Without the mapping
they would be written inside `docker.img`.

## Existing installations

This template's defaults describe a fresh AIO installation. Changing the image
from `:latest` to `:aio` does **not** migrate an existing separate PostgreSQL
database or move the old `/var/lib/silo` appdata into `/config`. Unraid also
retains saved container settings, so an existing container can still have its
old image, mappings, and connection URLs after the published template changes.

Back up the database, application data, saved template, and `SECRET_KEY` before
planning a migration. Keep the old service containers and data until the
migration has been completed and verified. Clearing `DATABASE_URL` starts a
bundled database; it does not import the external one. The separate
`Silo-PostgreSQL` template remains available for installations that need it.

The test behind these defaults covered a **fresh install**, not migration:
Unraid 7.4.0-rc.1, Silo build 1811, bundled PostgreSQL and Redis, setup and
library persistence across a restart, a synthetic H.264/AAC sample, 720p direct
play, and 480p hardware VA-API transcoding with pause, seek, and resume.
The updated template was also imported into Unraid and tested with Redis off,
then Meilisearch on while `ENABLE_REDIS=false`, then Meilisearch off again.
Readiness passed in each state. The service connection checks passed with
Meilisearch enabled, which automatically started bundled Redis; disabling
Meilisearch returned search to PostgreSQL and stopped the now-unneeded Redis.
The test instance was restored to Redis on and Meilisearch off. External
services and NVIDIA transcoding were not exercised.

## Hardware transcoding

Hardware acceleration is optional. CPU-only hosts need no GPU parameters.

For Intel or AMD, if `/dev/dri` exists on the host, add
`--device=/dev/dri:/dev/dri` to **Extra Parameters**, keeping the existing
restart, shutdown, and memory options. Do not add an empty Device entry:
DockerMan renders it as the invalid argument `--device=''`. Silo detects the
passed-through device; hardware VA-API transcoding was verified in the fresh
AIO test described above.

For NVIDIA NVENC/NVDEC:

1. Install the **Nvidia-Driver** plugin from Community Applications and reboot if prompted.
2. Open the plugin settings and copy the desired GPU UUID.
3. Add `--runtime=nvidia` to **Extra Parameters**, keeping the existing options.
4. Enter the UUID in **NVIDIA GPU UUID** (or use `all`). Leave
   `NVIDIA_DRIVER_CAPABILITIES` set to `compute,video,utility`.

Silo's `auto` hardware-acceleration mode detects an exposed NVIDIA device and
selects NVENC. The template does not force a GPU device or runtime, so it can
also start on hosts without one.

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
