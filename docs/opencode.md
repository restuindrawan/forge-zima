# OpenCode

- OpenCode v2 has no `web` command. This app runs `opencode serve --service` on `0.0.0.0:4096`, with remote access enabled for OpenTunnel.
- Locally, open `http://<zima-ip>:4096`. Do not forward port `4096` on your router.
- From outside, run this in the container (ZimaOS container terminal, or SSH) and open the one-time link it prints. Links work once and expire within minutes:

```bash
opencode pair --remote
```

- Set `OPENCODE_SERVER_PASSWORD` at install; it is saved as the service password. If left empty, a random password is generated in the config folder. Read it with `opencode service get password`.
- On first start the tunnel setup can take a minute. If it fails, the app still serves on the LAN and the container log tells you to retry the pair command.
