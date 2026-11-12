# Agent Instructions

This repository is a customized manual Docker Compose deployment of Nextcloud AIO. It is not the upstream Nextcloud AIO source tree.

## Update Requests

When the user asks to "update this repo", "pull the latest", or "do a diff" in this repository, do not assume they mean `git pull`.

The expected workflow is to compare this customized stack against upstream Nextcloud AIO manual install files:

1. Resolve the release to pin and the upstream commit its images were built from, then fetch manual install files at that commit. Do not review upstream `main`: it can carry compose changes for images that are not released yet.
   ```bash
   eval "$(scripts/aio-release)"   # sets AIO_IMAGE_TAG, UPSTREAM_AIO_SHA, NEXTCLOUD_VERSION
   curl -fsSL -o /tmp/nextcloud-aio-latest.yml "https://raw.githubusercontent.com/nextcloud/all-in-one/${UPSTREAM_AIO_SHA}/manual-install/latest.yml"
   curl -fsSL -o /tmp/nextcloud-aio-sample.conf "https://raw.githubusercontent.com/nextcloud/all-in-one/${UPSTREAM_AIO_SHA}/manual-install/sample.conf"
   ```
2. Diff upstream Compose against the local stack:
   ```bash
   diff -u compose.yaml /tmp/nextcloud-aio-latest.yml
   ```
3. Diff upstream environment variables against the tracked example:
   ```bash
   diff -u .env.example /tmp/nextcloud-aio-sample.conf
   ```
4. Report changes that may need manual adoption, especially new or removed environment variables, service changes, volume changes, ports, healthchecks, image behavior, or changed defaults.
5. Set every `ghcr.io/nextcloud-releases/aio-*` image to `${AIO_IMAGE_TAG}`. If making any repo edit as part of the sync, add or update `UPSTREAM-SYNC.md` with the exact `${UPSTREAM_AIO_SHA}` and `${AIO_IMAGE_TAG}`, source files reviewed, changes adopted, and intentional non-adoptions.

Only run `git pull` if the user explicitly asks to update the Git checkout itself. The remote may require user-controlled SSH hardware-key authentication, so do not treat a failed `git pull` as part of the normal AIO update workflow.

## Local Customizations To Preserve

- `compose.yaml` is heavily customized for ipvlan networking and an external reverse proxy.
- Apache uses ipvlan for reverse proxy access and the internal bridge for backend services.
- Talk is ipvlan-only.
- Most other services are bridge-only.
- Arcane image updates are disabled and should stay disabled for this stack.
- The memories transcoder customization should be preserved.
- `.env` is ignored because it contains deployment-specific values. Do not read, edit, or ask to commit `.env` unless the user explicitly requests it. Use `.env.example` for tracked variable comparisons.

## Operational Caution

Do not auto-update container images for this stack. AIO images are pinned to one dated release tag and move only in a reviewed sync; keep Renovate disabled for them. Upstream AIO image changes can require matching Compose or environment changes, so the diff review must happen before any `docker compose pull` or restart.
