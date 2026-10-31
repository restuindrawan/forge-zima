# forge-zima

Personal community app store for [ZimaOS](https://github.com/IceWhaleTech/ZimaOS), using the ZimaOS App Store v2 protocol.

## Add this store to ZimaOS

1. Open ZimaOS → **App Store** → **⋯ / Settings** → **Add Source**.
2. Paste the store URL:

```text
https://restuindrawan.github.io/forge-zima/store.json
```

3. Refresh the App Store. The apps below should appear.

## Apps

| App | Category | Image | Description |
| --- | --- | --- | --- |
| Seanime | Media | `umagistr/seanime:v3.10.4` | Default image, runs as root, no UID/GID matching needed, port `3212` |
| Seanime Rootless | Media | `umagistr/seanime:v3.10.4-rootless` | Self-hosted anime and manga media server (non-root, UID 1000), port `3211` |
| Seanime HwAccel | Media | `umagistr/seanime:v3.10.4-hwaccel` | Non-root, Jellyfin-FFmpeg with Intel QSV/VAAPI via `/dev/dri`, amd64 only, port `3213` |
| Seanime CUDA | Media | `umagistr/seanime:v3.10.4-cuda` | Non-root (UID 1001), NVENC/NVDEC via the `nvidia` runtime, amd64 only, port `3214` |
| OpenCode | Developer | `ghcr.io/anomalyco/opencode:2.2.5` | Open source AI coding agent (`serve --service`) on port 4096, with OpenTunnel pair links for outside access. Set `OPENCODE_SERVER_PASSWORD` when installing. |

## Seanime variants

Seanime ships as four separate store apps, one per image variant from [`umag/seanime-docker`](https://github.com/umag/seanime-docker/tree/main/examples), so you can install them side by side and compare. Each has its own app ID, config folder under `/DATA/AppData/<app-id>/config`, and host port, so settings are not shared. Do not point two variants at the same downloads folder at the same time.

| Variant | Pick it when | Notes |
| --- | --- | --- |
| Default | Media ownership is messy and you want it to just work | Runs as root. Files in downloads end up root-owned. |
| Rootless | You want the safe option | Runs as UID 1000. Folders must be writable by that user. |
| HwAccel | The box has an Intel GPU and you transcode | Set `VIDEO_GID`/`RENDER_GID` from `ls -ln /dev/dri`. |
| CUDA | The box has an NVIDIA GPU with the Container Toolkit | Runs as UID 1001. Needs the `nvidia` Docker runtime. |

Upstream says the hardware variants could not be tested in CI, so verify GPU transcoding on your own hardware. All four apps are pinned to `v3.10.4`, so bump them by hand.

## Seanime notes

- The Rootless app is based on the [rootless example](https://github.com/umag/seanime-docker/tree/main/examples/02-rootless) from `umag/seanime-docker`. The other variants follow the same pattern, but the config path differs: `/root/.config/Seanime` for Default, `/home/seanime/.config/Seanime` for the rest.
- The rootless and hwaccel apps run as the non-root user `seanime` (UID/GID `1000`), and CUDA runs as `1001`. The config, anime and downloads folders must be readable and writable by that user.
- Default volumes: config at `/DATA/AppData/$AppID/config`, anime at `/DATA/Media/Anime`, downloads at `/DATA/Media/Downloads`. Change them during install if your library lives elsewhere.
- WebUI ports: `3211` (Rootless), `3212` (Default), `3213` (HwAccel) and `3214` (CUDA) on the host, forwarding to `43211` in the container.

### Publishing Seanime outside your LAN

Seanime's [config docs](https://seanime.app/docs/config) require a server password when the server is reachable from outside the machine. No password is shipped in this public repo. Pick one option, then restart Seanime:

1. During install, set `SEANIME_SERVER_PASSWORD` if the install form shows it. It is passed as `--password` and defaults to empty.
2. Or after the first start, edit `config.toml` in the config folder (`/DATA/AppData/<app-id>/config/`) and set this under `[server]`, then restart:

```toml
[server]
password = 'use-a-long-random-password'
```

A password is not encryption. Prefer a VPN such as Tailscale or an HTTPS reverse proxy. Avoid forwarding plain HTTP port `3211` on your router.

## OpenCode notes

- OpenCode v2 has no `web` command. This app runs `opencode serve --service` on `0.0.0.0:4096`, with remote access enabled for OpenTunnel.
- Locally, open `http://<zima-ip>:4096`. Do not forward port `4096` on your router.
- From outside, run this in the container (ZimaOS container terminal, or SSH) and open the one-time link it prints. Links work once and expire within minutes:

```bash
opencode pair --remote
```

- Set `OPENCODE_SERVER_PASSWORD` at install; it is saved as the service password. If left empty, a random password is generated in the config folder. Read it with `opencode service get password`.
- On first start the tunnel setup can take a minute. If it fails, the app still serves on the LAN and the container log tells you to retry the pair command.

## Repository structure

```text
forge-zima/
├── Apps/
│   ├── Seanime/                 # Default (root) variant
│   │   ├── docker-compose.yml   # Compose + top-level x-casaos metadata
│   │   └── icon.png
│   ├── Seanime-Rootless/        # Same layout as above
│   ├── Seanime-HwAccel/         # Same layout as above
│   ├── Seanime-CUDA/            # Same layout as above
│   └── OpenCode/
│       ├── docker-compose.yml
│       └── icon.png
├── store-config.json            # Store identity (v2)
├── supported-languages.json
├── category-list.json
└── .github/workflows/
    ├── validate.yml             # Builds dist/ on pull requests
    └── release.yml              # Builds and deploys dist/ to gh-pages on push to main
```

Edit only the source files. `dist/` is generated by [IceWhaleTech/build-appstore-action](https://github.com/IceWhaleTech/build-appstore-action) and published to the `gh-pages` branch. Do not commit `dist/`.

## Add a new app

1. Create `Apps/<AppName>/docker-compose.yml`.
2. Give the project a top-level `name:` and add a top-level `x-casaos:` block. Required fields: `id` (reverse-domain, e.g. `app.seanime`), `main`, `index`, `port_map` (quoted string), `icon`, `title.en_US`, `category`, and `version`. See the [compose and x-casaos spec](https://github.com/IceWhaleTech/CasaOS-AppStore/blob/main/docs/specs/compose-and-x-casaos.md).
3. Add `Apps/<AppName>/icon.png` and point `x-casaos.icon` at it.
4. Validate locally, then open a pull request:

```bash
docker compose -f Apps/<AppName>/docker-compose.yml config -q
```

5. Merge to `main`. The release workflow rebuilds and redeploys the store.

## License

MIT. App definitions are packaging only. Each app remains under its upstream license, for example Seanime is GPL-3.0.
