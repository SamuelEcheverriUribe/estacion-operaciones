# Infra: Contenedores (Docker) — plan de despliegue gauzy

> Estado: IMPLEMENTADO (16-sep) — Docker instalado y demo gauzy corriendo.
> Definición del flujo + resultado real (ver §7-§8).

## 1. Por qué contenedores aquí

- La estación (gauzy: API NestJS + web Angular + Postgres) se monta de forma
  reproducible: misma infra en casa que en un servidor futuro.
- Aislamiento: nada de "instala en el sistema", todo dentro de imágenes.
- Aprendizaje: compose, volúmenes, redes, healthchecks → portátil a cualquier
  cliente (Conecty).

## 2. Stack objetivo (dev, liviano)

| Servicio | Imagen | Nota |
|---|---|---|
| API gauzy | ghcr.io/ever-co/gauzy-api | NestJS |
| Web UI | ghcr.io/ever-co/gauzy-webapp | Angular (nginx) |
| DB dev | SQLite embebida | sin servicio externo (modo dev) |
| DB prod | postgres:16 + redis | cuando se requiera escala |

> En dev se arranca API+Web con SQLite para aprender sin infra pesada.
> Prod/demo de gauzy trae `docker-compose.demo.yml` (imágenes prebuilt).

## 3. Mapa de conceptos (aprendizaje)

- **Imagen** = receta (código + dependencias + runtime).
- **Contenedor** = imagen corriendo, aislado.
- **Compose** = orquestar varios contenedores en un `docker-compose.yml`
  (servicios, redes, volúmenes, env).
- **Volumen** = datos persistentes fuera del contenedor (la DB vive aquí,
  sobrevive al `docker compose down`).
- **Red** = comunicación interna entre servicios (api→db) sin exponer todo al
  host.
- **Puerto** = entrada desde el host: `4200` (web), `3000/api` (API)…
- **Env** = configuración fuera del código (`.env`, no al repo).
- **Healthcheck** = prueba de que el servicio está vivo.

## 4. Comandos núcleo (para la operación diaria)

```bash
docker compose pull                       # trae imágenes
docker compose up -d                       # levanta en segundo plano
docker compose ps                          # estado
docker compose logs -f <servicio>          # logs en vivo
docker compose down                        # apaga (sin borrar volúmenes)
docker compose down -v                     # apaga Y borra volúmenes (¡datos!)
docker exec -it <contenedor> sh            # terminal dentro de un contenedor
```

## 5. Seguridad

- `.env` con credenciales NUNCA se commitea (`.gitignore`). Solo `.env.example`.
- Puertos solo en local (127.0.0.1) salvo que se decida exponer vía Tailscale.
- Datos de clientes (Suiza/HITL): no a la nube pública; esta estación es local
  (Tailscale para acceso remoto propio).
- Actualizar imágenes con `docker compose pull` (no `:latest` productivo, fijar
  tags).

## 6. Plan de ejecución (F2)

1. Instalar Docker Engine (sudo dnf) + plugin compose.
2. Probar `docker run hello-world` y un contenedor de ejemplo (nginx).
3. `docker compose -f docker-compose.demo.yml up` → gauzy en 4200
   (admin@ever.co / admin).
4. Ejercicio de aprendizaje: mapear puertos, subir bajada, volúmenes.
5. Documentar aquí el resultado real (restricciones, puertos, comandos útiles).

## 7. Resultado real (16-sep-2026, implementado)

- Docker Engine 29.8.1 + Compose v5.5.1 instalados (dnf, repos oficiales) en
  Fedora. `bark` en grupo `docker` (efectivo en próxima sesión; mientras se usa
  sudo con clave de sesión diaria, autorización de Samuel 16-sep).
- Repo gauzy clonado shallow en `~/Proyectos/ever-gauzy`.
- `docker compose -f docker-compose.demo.yml up -d` → 3 contenedores:
  - `db` postgres:17-alpine (healthy) → 5432, volumen `ever-gauzy_postgres_data`
  - `api` ghcr.io/ever-co/gauzy-api → 3000/api (HTTP 200)
  - `webapp` ghcr.io/ever-co/gauzy-webapp (nginx) → 4200 (HTTP 200)
- **Login demo**: http://localhost:4200 — admin@ever.co / admin.
- Red `ever-gauzy_overlay` (bridge) interna; puertos mapeados al host.
- Comandos propios: `sudo docker compose -f docker-compose.demo.yml ps/logs/down`
  (ver §6; `docker` directo tras re-login).

> Próximo (F3): entrar por la UI, crear un tenant, y mapear contactos/props.

## 8. API real — contrato aprendido (16-sep)

### 8.1 Autenticación (idéntica para móvil / scripts)
1. `POST /auth/login` `{email, password}` → responde `token` (JWT) y
   `user.tenantId`.
2. Toda petición posterior: `Authorization: Bearer <token>` **+** header
   `Tenant-Id: <tenantId>` (sin él → 403 multi-tenant).
3. `GET /user/me` devuelve el perfil (rol SUPER_ADMIN en demo).

### 8.2 Esquema `organization-contact` (clientes/leads)
- Campos de creación (DTO): `organizationId` (UUID, obligatorio),
  `name`, `primaryEmail`, `primaryPhone`, `contactType`, `notes`, `budget`,
  `budgetType`, `imageId`.
- `contactType` ∈ `CLIENT | CUSTOMER | LEAD`.
- `GET /organization-contact` sin `where` **sí** funciona (devuelve 146 demo);
  otras rutas (`/organization`, `/employee`) exigen `where` no vacío.

### 8.3 Rate-limit (importante para automatizar)
- El **login está throttleado**: varias llamadas seguidas → `HTTP 429`.
  Mitigación: cachear el token (`/tmp/opencode/gauzy_token`) y reintentar con
  backoff; no loguearse por cada operación. (Implementado en `gauzy-sync`.)

### 8.4 Bridge construido
- **`~/.local/bin/gauzy-sync`** (Python, +x): traduce `contactos.json` y
  `estado.json` a entidades gauzy. Dry-run por defecto (sin red) y
  `--apply` (idempotente por `name`). Mapeo en `docs/flujos/mapeo-gauzy.md`.
- **`~/.local/bin/gauzy-cli`** (Python, +x): cliente mínimo (`ping`, `login`,
  `contacts`) — mismo contrato que consumirá la app móvil.