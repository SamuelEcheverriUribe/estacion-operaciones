# Estrategia de build del fork (documentación-primero)

> Decisión de Samuel (16-sep): montar gauzy desde SU fork, no del repo original.
> Este documento fija cómo construir y desplegar imágenes propias, qué riesgo
> existe y cuál es la ruta recomendada.

## 1. Estado actual

| Pieza | Estado |
|---|---|
| Fork `SamuelEcheverriUribe/ever-gauzy` | ✅ creado (parent ever-co/ever-gauzy) |
| Clon local | ✅ origin=fork, upstream=original |
| Demol corriendo | imágenes `ghcr.io/ever-co/gauzy-*:latest` (del repo ORIGINAL) |
| Docker local | ✅ 29.8.1 + compose v5.5.1 |
| Dockerfiles | `~/.deploy/api/Dockerfile`, `~/.deploy/webapp/Dockerfile` (BuildKit, `--mount` bind) |

## 2. Cómo compila gauzy sus imágenes (lo aprendido)

- Dockerfiles con **multi-stage + BuildKit** (`RUN --mount`): node_modules en
  bind, no en capas → imágenes livianas.
- **`VERDACCIO_TOKEN` es OPCIONAL** (verificado en `.deploy/api/Dockerfile`
  líneas 120-145): el registro privado `packages.ever.co` **solo se usa si** se
  pasan `VERDACCIO_REGISTRY` o `VERDACCIO_TOKEN`. Sin ellos → instala del
  registro **público** (`registry.yarnpkg.com`, el `.npmrc` del repo ya lo fija).
  El `yarn.lock` tiene **0** referencias a `packages.ever.co`. → **El build del
  fork NO requiere token privado.** (Riesgo #1 descartado.)
- `NX_NO_CLOUD=true` deja Nx Cloud inerte; el build corre local.
- Apps a construir: API (`apps/api`), Webapp (`apps/gauzy`, nginx), y opcional
  desktop/agent/mcp.

## 3. Rutas posibles (y por qué)

### A. CI/CD en tu fork: Actions → GHCR (RECOMENDADA)
1. En el fork, crear `.github/workflows/build-own.yml`:
   - se dispara al push a tu rama `estacion` (o `develop`).
   - `docker/build-push-action` buildx → publica a `ghcr.io/SamuelEcheverriUribe/gauzy-api` y `.../gauzy-webapp` (alimentado con `GITHUB_TOKEN`).
2. El `docker-compose.demo.yml` propio apunta a tus imágenes (no a ever-co).
3. Ventajas: build en la nube de GitHub (gratis, sin consumir tu PC), CI/CD
   real (aprendizaje exacto Conecty), imágenes reproducibles.
4. Condición: sincronizar con upstream (fork se mantiene al día con
   `git fetch upstream` + merge) para no quedarte solo con un snapshot.

### B. Build local desde el fork
1. `cd ~/Proyectos/ever-gauzy && yarn bootstrap` (instala deps, pesado).
2. `docker buildx build . -f .deploy/api/Dockerfile -t ghcr.io/SamuelEcheverriUribe/gauzy-api:dev` ...
3. Tiempo y RAM altos; sin Actions. Útil para hacer cambios de código y probar
   en el acto.

### C. Mixta (recomendada para arrancar)
- **Build local puntual** para validar que los cambios de código funcionan
  (rápido, detección temprana en tu máquina).
- **Actions→GHCR** como ruta definitiva de publicación (cuando el rework sea
  real y la demo esté validada).

## 4. Riesgos y mitigaciones

| Riesgo | Mitigación |
|---|---|
| ~~Sin `VERDACCIO_TOKEN` no build~~ | **Descartado**: sin token usa el registro público (ver §2). |
| Fork queda atrás de upstream | Flujo `estacion` + `git fetch upstream && git merge` programado; PRs propios. |
| Imágenes originales eternas en compose | Compose propio `docker-compose.own.yml` (ya creado) apuntando a GHCR tuyo. |
| Secrets en ambiente | `.env` nunca al repo; `GITHUB_TOKEN` con permiso `packages:write` (scoped). |
| Build lento en PC (RAM/CPU) | Preferir Actions→GHCR; build local solo para validar cambios puntuales. |

## 5. Próximo paso concreto (para CUANDO Samuel valide la demo)

1. `git fetch upstream` en el clon local → merge develop en rama `estacion`.
2. Probar **build local de la API** (medir tiempo/RAM) → documentar resultado.
3. Si el build local funciona → armar Actions en el fork (build-own.yml) →
   apuntar compose a GHCR de Samuel.
4. `DEMO=false` + seed limpio → primer tenant propio (Samuel) → mapear datos
   reales (flujos/mapeo-gauzy.md).

> No ejecutar aún: pendiente de exploración/validación de Samuel (está viendo
> la UI). Este doc queda como el plan de ejecución.

## 6. Preparación ya hecha (16-sep, rama `estacion` del fork)

- Rama **`estacion`** creada y pusheada a `origin`
  (`SamuelEcheverriUribe/ever-gauzy`). Se creó desde `develop`.
  - No dispara ningún workflow automático (los `push:` del repo están limitados
    a otras ramas: develop, stage, droplets, apps…), verificado.
- **`.github/workflows/build-own.yml`**: build propio de API + Webapp →
  `ghcr.io/SamuelEcheverriUribe/gauzy-{api,webapp}`. **Solo manual**
  (`workflow_dispatch`) con input `demo` y `publish`, más caché `type=gha`.
  Sin secretos privados (usa registro público, ver §2).
- **`docker-compose.own.yml`**: copia del demo apuntando a las imágenes del
  fork (GHCR de Samuel), no a las de ever-co.
- Validado: YAML OK; el compose referencia `ghcr.io/SamuelEcheverriUribe/...`.

### Cómo se usará (cuando se decida)
```
# 1) GitHub → Actions → "Build imágenes propias (estación)" → Run workflow
# 2) traer y levantar:
sudo docker compose -f ~/Proyectos/ever-gauzy/docker-compose.own.yml pull
sudo docker compose -f ~/Proyectos/ever-gauzy/docker-compose.own.yml up -d
```
> Al ser imágenes del fork, se puede modificar el código (p. ej. vista
> "Estación freelance") y reconstruir con el botón, sin tocar el upstream.