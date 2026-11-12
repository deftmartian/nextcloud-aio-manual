# Upstream AIO Sync Log

This file records the exact `nextcloud/all-in-one` upstream commit used when
manually syncing this customized stack. The local compose file intentionally
preserves ipvlan networking, external reverse-proxy handling, Arcane labels,
`TURN_DOMAIN` for a public TURN host other than `NC_DOMAIN`, and
the memories transcoder customization.

## 2026-11-12 (later) - HaRP adopted from `97ef10c`, images `20261109_101109`

Changes adopted:

- The `nextcloud-aio-harp` service from the v15.0.2 `latest.yml`, on
  `nextcloud-internal` with a container name and the Arcane label, and the
  Docker socket mounted from `/var/run/docker.sock` instead of
  `${WATCHTOWER_DOCKER_SOCKET_PATH}`. This supports Recognize 13's
  `recognize_backend` ExApp on the GPU.
- `HP_EXAPP_BIND_MOUNTS_ENFORCE_ALL=true`, from upstream `main` (not in the
  v15.0.2 file). The v15.0.2 HaRP image already supports it.
- Apache's optional `depends_on` HaRP, Nextcloud's `HARP_ENABLED` and
  `HP_SHARED_KEY`, and the `nextcloud_aio_harp` volume.

Not adopted:

- No compose profile, as for the other services.

The deploy daemon is registered by hand; AIO does not register it.

## 2026-11-12 - `nextcloud/all-in-one@97ef10c8049d307a97ad7ed3f2534f00b97f2133`, images `20261109_101109`

Source files reviewed:

- `manual-install/latest.yml`
- `manual-install/sample.conf`

AIO v15.0.2. The images tagged `20261109_101109` were built from `97ef10c`
(the v15.0.2 version bump) and **ship Nextcloud 35.0.2, a major upgrade from
34.0.5**. Both `manual-install` files at `97ef10c` are identical to the
`e525bb3` sync. Every field of every shared service and every
`sample.conf` variable was compared again, not only the change since the last
sync.

Changes adopted:

- Every `ghcr.io/nextcloud-releases/aio-*` image is pinned to
  `20261109_101109`, and go-vod is pinned by digest. With `:latest`, any merge
  (Arcane redeploys on each one) could pull a new release.
  `scripts/aio-release` prints the tag to pin, the upstream commit its images
  were built from, and the Nextcloud version. Renovate is disabled for the AIO
  images.
- The Nextcloud service passes `COLLABORA_ENABLED` and `ONLYOFFICE_ENABLED`
  through from `.env`, as upstream does. Since the first commit they were
  hardcoded to `true` and `false`. The entrypoint enables Collabora only on
  the exact string `yes`, so `true` skipped the `richdocuments` install and
  update, `wopi_url`, the WOPI allowlist, and `richdocuments:activate-config`.
  With `REMOVE_DISABLED_APPS=yes` it also removed `richdocuments` on every
  start. The deployment `.env` must have `COLLABORA_ENABLED="yes"`.

Not adopted:

- Upstream `main` (`6a92d77`) adds a notify-push `/tmp` tmpfs and CA mount,
  fulltextsearch `init: true`, and HaRP's `HP_EXAPP_BIND_MOUNTS_ENFORCE_ALL`.
  None is in a released image yet; adopt them with the release that ships
  them. Leave out the CA mount while `NEXTCLOUD_TRUSTED_CACERTS_DIR` is empty,
  because an empty source makes the volume spec invalid.
- The `NEXTCLOUD_MOUNT` bind mount and the CA mounts on nextcloud and talk.
  Both variables are empty here.
- Every non-adoption from the 2026-09-30 sync still applies.

## 2026-11-03 - `nextcloud/all-in-one@e525bb35e57a106fa628caafacd2187223e858d1`

Source files reviewed:

- `manual-install/latest.yml`
- `manual-install/sample.conf`

Covers AIO v14.4.3, the second part of the brotli security rebuilds. It still
ships Nextcloud 34.0.5. Neither file changed upstream since `70c9331`, at either
the `v14.4.3` tag or `main`.

Changes adopted:

- None. This entry records the sync so that the redeploy pulls the rebuilt
  `:latest` images.

## 2026-10-28 - `nextcloud/all-in-one@70c9331e3a676c1d3bce796cc090b3927afcc445`

Source files reviewed:

- `manual-install/latest.yml`
- `manual-install/sample.conf`

Covers AIO v14.4.1 and v14.4.2. v14.4.2 only rebuilds images for the brotli
advisories and still ships Nextcloud 34.0.5.

Changes adopted:

- PostgreSQL mounts `/var/lib/postgresql` as tmpfs (upstream `2bd2e2b`). The
  image declares that path as a volume, so without the tmpfs each recreate
  leaves an orphaned anonymous volume. The named data volume at
  `/var/lib/postgresql/data` is unaffected.

`sample.conf` is unchanged since `f294364`. Every non-adoption from that sync
still applies.

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
