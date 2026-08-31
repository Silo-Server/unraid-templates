# Contributing to the Silo Unraid Templates

The [Silo contribution guide](https://github.com/Silo-Server/.github/blob/main/CONTRIBUTING.md)
covers project-wide coordination, focused changes, evidence, AI disclosure, and
pull request expectations. Those requirements apply here; this guide adds the
Unraid-specific workflow.

## Before you start

Open an [issue](https://github.com/Silo-Server/unraid-templates/issues) before
changing container topology, required services, networking defaults, storage
layout, or upgrade behavior. Documentation corrections and narrow template
fixes can go straight to a pull request.

This repository owns Community Applications XML and Unraid-specific guidance.
Server configuration and container behavior belong in
[`silo-server`](https://github.com/Silo-Server/silo-server).

## Validate your change

Validate every XML file before opening a pull request:

```sh
xmllint --noout ca_profile.xml templates/*.xml
```

For template behavior changes, install the templates on an Unraid host and
verify a fresh setup with PostgreSQL, Redis, and Silo. Confirm paths,
placeholders, ports, permissions, dependency startup order, and any hardware
device settings. Include the tested Unraid version and actual results in the
pull request.

Do not commit credentials, generated Unraid user templates, appdata, database
files, or host-specific paths.

## Open the pull request

Use a Conventional Commit title, explain the install or upgrade impact, and
paste the actual validation results. Read the
[AI-assisted contribution policy](https://github.com/Silo-Server/silo-server/blob/main/docs/ai-contributions.md)
and include its disclosure block.
