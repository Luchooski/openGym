# Deploy en Railway

Dos servicios del mismo repo (`api` y `web`) en un proyecto de Railway, con un volumen para `/data`.
Solo `web` es público; `api` queda en la red privada.

## 1. Servicio `api`

- **New → GitHub Repo** → `Luchooski/openGym`. Renombrar el servicio a `api` (el nombre define el
  host privado `api.railway.internal`).
- Settings → **Root Directory**: `/api` (usa `api/Dockerfile`, target `default`).
- **Volume**: montarlo en `/data`. Ahí viven usuarios, passkeys, entrenamientos y el secret de sesión.
- Sin dominio público.
- Variables:

| Variable | Valor |
| --- | --- |
| `PORT` | `3000` |
| `DATA_DIR` | `/data` |
| `RP_ID` | dominio de `web`, sin `https://` (ej. `opengym-production.up.railway.app`) |
| `ORIGIN` | `https://` + ese dominio |
| `FIRST_USER_ADMIN` | `1` (el primer perfil creado queda como admin) |
| `TRUST_PROXY` | `1` |

## 2. Servicio `web`

- **New → GitHub Repo** → mismo repo. Renombrar a `web`.
- Settings → Root Directory vacío (contexto = raíz del repo).
- Variable `RAILWAY_DOCKERFILE_PATH` = `web/Dockerfile`.
- **Networking → Generate Domain**, puerto `8080`. Copiar ese dominio a `RP_ID`/`ORIGIN` de `api`.
- Variables:

| Variable | Valor |
| --- | --- |
| `NGINX_PORT` | `8080` |
| `BACKEND` | `api.railway.internal` |
| `PORT` | `3000` (puerto de la API, no de nginx) |
| `RESOLVER` | `[fd12::10]` (DNS interno de Railway) |
| `RESOLVER_IPV6` | `on` (la red privada de Railway es IPv6) |

## 3. Después del primer deploy

1. Abrir el dominio de `web` y crear el perfil con passkey: ese queda como admin.
2. En `api`, poner `INVITE_ONLY=1` para que nadie más se registre sin código. Redeploy.
3. Backups: el volumen de Railway tiene backups desde la UI del volumen. Además, en la app,
   Settings permite exportar todo a JSON.

## Notas

- `RP_ID` y `ORIGIN` tienen que coincidir exactamente con el dominio con el que entrás. Si después
  ponés un dominio propio, cambialos y los passkeys creados con el dominio viejo dejan de servir.
- Si `/api` responde 502 o queda colgado, revisar que `BACKEND`, `RESOLVER` y `RESOLVER_IPV6`
  estén como arriba y que el servicio se llame `api`.
- Actualizar desde el original: `git fetch upstream && git merge upstream/main && git push`.
