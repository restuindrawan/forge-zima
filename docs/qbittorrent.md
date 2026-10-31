# qBittorrent

Uses the [linuxserver.io image](https://github.com/linuxserver/docker-qbittorrent), pinned to `5.3.0`.

- WebUI on host port `8181`. The user is `admin` and a temporary password is printed in the container log on every start until you set your own in **Tools > Options > WebUI**.
- `WEBUI_PORT` is `8181` on both sides of the port mapping. linuxserver notes that a remapped WebUI port causes CSRF errors, so if you change the port, change `WEBUI_PORT` and the published port together.
- Incoming BitTorrent connections use `6881` (TCP and UDP). Forward it on your router for better connectivity. Do not forward the WebUI port.
- Volumes: config at `/DATA/AppData/<app-id>/config`, anime at `/DATA/Media/Anime` (as `/anime`) and downloads at `/DATA/Media/Downloads` (as `/downloads`).
- Files are written as `PUID`/`PGID` (default `1000`). That user must be able to write to the mounted folders.

## Using it with Seanime

Seanime passes its own destination path to qBittorrent as the save path, so qBittorrent has to see the same folders under the same paths. That is why this app mounts `/anime` and `/downloads` like the Seanime apps do. If you change either path here, change it in Seanime too.

In Seanime's torrent client settings, use the box's address and port `8181`.
