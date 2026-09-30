# Upstream AIO Sync Log

This file records the exact `nextcloud/all-in-one` upstream commit used when
manually syncing this customized stack. The local compose file intentionally
preserves ipvlan networking, external reverse-proxy handling, Arcane labels, and
the memories transcoder customization.

## 2026-09-30 - `nextcloud/all-in-one@f294364355190ddedcd646c471a0bfe852173d47`

Source files reviewed:

- `manual-install/latest.yml`
- `manual-install/sample.conf`

Changes adopted:

- Apache and Talk tmpfs now mount `/run` instead of `/var/log/supervisord` and `/var/run/supervisord`.
- ClamAV sets `init: true`, mounts `/run`, and drops `/run/clamav` plus the supervisord tmpfs entries.
- Collabora healthcheck probes with `/usr/bin/coolwsd --probe --use-env-vars`.
- Collabora `extra_params` sets `logging.level` and `logging.level_startup` from `COLLABORA_LOG_LEVEL`.
- `.env.example` documents `COLLABORA_LOG_LEVEL=warning`. The deployment `.env` needs the same key before the next recreate.
- `ADDITIONAL_COLLABORA_OPTIONS` is one coolwsd argument. The old bracket-list form was interpreted by the removed start.sh wrapper, and the current image exits 70 if it receives that string.
- Apache gets `HARP_HOST=nextcloud-aio-harp` so the new Caddyfile can load its `/exapps` route. The route has no running backend.

Not adopted:

- The HaRP service, its volume, Apache `depends_on`, Nextcloud `HARP_ENABLED` and `HP_SHARED_KEY`, and `WATCHTOWER_DOCKER_SOCKET_PATH` stay omitted. A `depends_on` for a service this file does not define would make Compose fail, and this stack does not run the App API deploy daemon. Unset `HARP_ENABLED` leaves `app_api` disabled.
- EuroOffice data volume stays omitted with EuroOffice.
- Upstream's new sample defaults `NEXTCLOUD_UPLOAD_LIMIT=1G` and `APACHE_MAX_SIZE=1073741824` stay at this stack's 16G values.
- `INSTALL_LATEST_MAJOR` quote-only sample change is not applied. Local value stays `no`.
- Upstream optional services `nextcloud-aio-eurooffice`, `nextcloud-aio-onlyoffice`, `nextcloud-aio-talk-recording`, and `nextcloud-aio-whiteboard` remain omitted.
- Local-only `nextcloud-aio-memories-transcoder` remains preserved, along with ipvlan, the external reverse proxy, and Arcane `updater=false` labels.

## 2026-07-16 - `nextcloud/all-in-one@a36d46987daa2b28110d9860e8bbee42d3da92a1`

Source files reviewed:

- `manual-install/latest.yml`
- `manual-install/sample.conf`

Changes adopted:

- No runtime Compose or environment-contract changes were required; both
  upstream manual-install files were unchanged from the previous review.
- Separated the online upstream audit from the maintenance window in the
  update procedure.
- Added a generic rollback-point checkpoint and concise post-update health
  checks without prescribing a backup implementation.
- Corrected the architecture diagram and environment comments to show that
  OnlyOffice is intentionally omitted and its toggle is not active.

Not adopted:

- Upstream optional services `nextcloud-aio-eurooffice`,
  `nextcloud-aio-onlyoffice`, `nextcloud-aio-talk-recording`, and
  `nextcloud-aio-whiteboard` remain omitted from `compose.yaml`.
- Local-only `nextcloud-aio-memories-transcoder` remains preserved.

## 2026-07-03 - `nextcloud/all-in-one@92cd622226c54c7333174f7c28315cbbbd16f952`

Source files reviewed:

- `manual-install/latest.yml`
- `manual-install/sample.conf`

Changes adopted:

- Added this upstream sync log so future edits record the exact upstream commit.
- Updated the repository workflow docs to fetch upstream manual files by commit
  SHA before comparing them against the local stack.
- No runtime Compose or environment-variable changes were required for the
  services currently used by this stack.

Not adopted:

- Upstream optional services `nextcloud-aio-eurooffice`,
  `nextcloud-aio-onlyoffice`, `nextcloud-aio-talk-recording`, and
  `nextcloud-aio-whiteboard` remain omitted from `compose.yaml`.
- Local-only `nextcloud-aio-memories-transcoder` remains preserved.
