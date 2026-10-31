# Seanime

Seanime ships as four separate store apps, one per image variant from [`umag/seanime-docker`](https://github.com/umag/seanime-docker/tree/main/examples). You can install them side by side and compare. Each has its own app ID, config folder under `/DATA/AppData/<app-id>/config`, and host port, so settings are not shared. Do not point two variants at the same downloads folder at the same time.

## Variants

| Variant | App ID | Image tag | Runs as | Host port | Config path in container |
| --- | --- | --- | --- | --- | --- |
| Default | `app.seanime` | `v3.10.4` | root | `3212` | `/root/.config/Seanime` |
| Rootless | `app.seanime-rootless` | `v3.10.4-rootless` | `1000:1000` | `3211` | `/home/seanime/.config/Seanime` |
| HwAccel | `app.seanime-hwaccel` | `v3.10.4-hwaccel` | `1000:1000` | `3213` | `/home/seanime/.config/Seanime` |
| CUDA | `app.seanime-cuda` | `v3.10.4-cuda` | `1001:1001` | `3214` | `/home/seanime/.config/Seanime` |

All host ports forward to `43211` in the container. Default volumes are the config at `/DATA/AppData/$AppID/config`, anime at `/DATA/Media/Anime` and downloads at `/DATA/Media/Downloads`. Change them during install if your library lives elsewhere.

## Picking one

| Variant | Pick it when | Notes |
| --- | --- | --- |
| Default | Media ownership is messy and you want it to just work | Files in downloads end up root-owned. |
| Rootless | You want the safe option | Config, anime and downloads folders must be readable and writable by UID 1000. |
| HwAccel | The box has an Intel GPU and you transcode | Set `VIDEO_GID` and `RENDER_GID` from `ls -ln /dev/dri`. amd64 only. |
| CUDA | The box has an NVIDIA GPU with the Container Toolkit | Needs the `nvidia` Docker runtime. Runs as UID 1001. amd64 only. |

Upstream says the hardware variants could not be tested in CI, so verify GPU transcoding on your own hardware.

## Versions

All four apps are pinned to `v3.10.4`. Bump the image tag and `x-casaos.version` in each `docker-compose.yml` by hand.

## Publishing outside your LAN

Seanime's [config docs](https://seanime.app/docs/config) require a server password when the server is reachable from outside the machine. No password is shipped in this public repo. Pick one option, then restart Seanime:

1. During install, set `SEANIME_SERVER_PASSWORD` if the install form shows it. It is passed as `--password` and defaults to empty.
2. Or after the first start, edit `config.toml` in the config folder (`/DATA/AppData/<app-id>/config/`) and set this under `[server]`, then restart:

```toml
[server]
password = 'use-a-long-random-password'
```

A password is not encryption. Prefer a VPN such as Tailscale or an HTTPS reverse proxy. Avoid forwarding the plain HTTP host port (`3211` to `3214`) on your router.
