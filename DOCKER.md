# Docker

Two setups: **prod** (baked builds, `docker-compose.yml`) and **dev** (hot reload, `docker-compose.dev.yml`). Same ports — run one at a time.

## Ports

| Service | URL |
|---------|-----|
| Web      | http://localhost:8080 |
| API      | http://localhost:5080 |

## Dev (hot reload)

```powershell
docker compose -f docker-compose.dev.yml up
```

- **Web** — `vite dev` with HMR. Edit `.vue` / JS / CSS → browser updates instantly.
- **API** — `dotnet watch`. Save a `.cs` file → recompiles and restarts on save.

Source folders are bind-mounted; `node_modules` and `bin`/`obj` live inside the container (Linux builds — don't mix with host builds).

First run downloads `node:24-alpine` and `mcr.microsoft.com/dotnet/sdk:10.0` and runs `npm install`.

## Prod

```powershell
docker compose up --build
```

- **API** — multi-stage .NET 10 build → `aspnet:10.0` runtime image.
- **Web** — `npm run build` → nginx serving static `dist/`.

No hot reload: rebuild after changes (`docker compose up --build`).

## Useful commands

```powershell
docker compose down                          # stop prod
docker compose -f docker-compose.dev.yml down # stop dev
docker compose logs -f                       # tail prod logs
docker compose -f docker-compose.dev.yml logs -f web-dev   # vite logs
docker compose -f docker-compose.dev.yml logs -f api-dev   # dotnet logs
```

## Note for frontend code

The browser must call the API at `http://localhost:5080` (published host port), not `api:8080` (container-internal). If the app uses an API base URL, make it configurable (env var / Vite proxy) when needed.
