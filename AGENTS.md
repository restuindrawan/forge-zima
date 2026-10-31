# AGENTS.md

ZimaOS App Store v2 repo: only compose files, JSON and docs. No app code, no build, no tests. Pushing to `main` publishes the live store, so commit locally and push only when asked.

## Verify

- Validate every app: `for a in Apps/*/; do docker compose -f $a/docker-compose.yml config -q; done`. It needs no Docker daemon. `AppID variable is not set` warnings are expected (ZimaOS injects `$AppID`).
- CI (`.github/workflows`) runs `IceWhaleTech/build-appstore-action`: `validate.yml` on PRs, `release.yml` on push to `main` builds `dist/` and deploys it to `gh-pages`. Never commit `dist/` or `.cache/`. It only reads `Apps/` plus the root JSON files, so `docs/` is safe.

## Seanime variants: do not confuse Default (root) with Rootless

Four store apps from `umagistr/seanime`. Default is the root image and carries **no suffix anywhere** (title, folder, app ID, project name, container).

| Folder | Title | App ID / container | Image | Host port |
| --- | --- | --- | --- | --- |
| `Apps/Seanime` | Seanime | `app.seanime` / `seanime` | `v3.10.4` (root) | 3212 |
| `Apps/Seanime-Rootless` | Seanime Rootless | `app.seanime-rootless` | `v3.10.4-rootless` | 3211 |
| `Apps/Seanime-HwAccel` | Seanime HwAccel | `app.seanime-hwaccel` | `v3.10.4-hwaccel` | 3213 |
| `Apps/Seanime-CUDA` | Seanime CUDA | `app.seanime-cuda` | `v3.10.4-cuda` | 3214 |

- Config path differs: Default mounts `/root/.config/Seanime`; the others mount `/home/seanime/.config/Seanime`. CUDA runs as UID 1001, not 1000.
- A permissions helper service and `PUID`/`PGID` were removed on purpose. Do not re-add them.
- HwAccel uses numeric `group_add` via `VIDEO_GID`/`RENDER_GID`, not group names.
- Pinned versions: to bump, update image tag, `x-casaos.version`, `update_at`, `release_notes`, the README table and `docs/seanime.md` in all four apps.

## OpenCode (`Apps/OpenCode`)

- OpenCode v2 has no `web` command; the app runs `opencode serve --service`. Image is pinned and `x-casaos.version` tracks it.
- The OpenTunnel identity at `/root/.local/share/opentunnel` must stay a persisted volume. Without it, every container recreation changes the remote address and forces re-pairing.
- To bump: image tag, the "OpenCode x.y.z" text in the description, `version`, `update_at`, `release_notes`, and the README table.

## Compose and metadata conventions

- Each app needs a top-level `name:` and `x-casaos:` with `id`, `main`, `index`, `port_map` (quoted string), `icon`, `title.en_US`, `category`, `version`.
- `id`, `name`, service name, `container_name`, `main` and published port must be unique across all apps. Keep ports in the table above.
- `x-casaos.icon` is a jsdelivr URL containing the folder name (`.../forge-zima@main/Apps/<Folder>/icon.png`). If you rename a folder, update it.
- Renaming an `id` makes ZimaOS treat it as a new app, so existing installs must be reinstalled.
- Use `$$` for shell variables inside compose `command:` blocks, and `/DATA/AppData/$AppID/...` for bind sources.
- The repo is public: never ship default passwords. Use `${VAR:-}` and document the variable.
- Set `update_at` manually to the current date when changing an app.
- `x-casaos.tagline` and `description` are verbatim upstream wording (project site or README), identical across the four Seanime variants. Put packaging details (image variant, ports, pairing steps) in `tips.before_install`, not in the description.

## Docs

Per-app notes go in `docs/<app>.md`. Keep `README.md` short (intro, add-store steps, apps table linking to docs, repo structure, how to add an app).
