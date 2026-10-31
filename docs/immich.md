# Immich

Pinned to `v3.3.5`. The compose follows Immich's own `docker-compose.yml` for that release: `immich-server`, `immich-machine-learning`, Valkey (`redis`) and Postgres with VectorChord (`database`). Image digests for Valkey and Postgres are copied from upstream, so bump them together with the version.

- WebUI and API on host port `2283`. Create the admin account on first visit.
- Uploads go to `/DATA/Gallery/immich` (mounted at `/data`). The database (`pgdata`) and the machine learning model cache (`model-cache`) are under `/DATA/AppData/<app-id>/`. Immich requires the database on local storage, not a network share.
- Immich needs at least 6GB of RAM and 2 CPU cores. The machine learning and import jobs are CPU heavy at first. This app runs machine learning on CPU only.
- `DB_PASSWORD` defaults to `postgres`, which is the value in Immich's `example.env`. The database has no published port, so it is only reachable inside the app's own network. Do not change it after the first start, because Postgres keeps the password in its data folder.
- Upgrading: read the Immich release notes first. Major versions can include breaking changes.
