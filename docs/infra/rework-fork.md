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
- **Secreto `VERDACCIO_TOKEN`**: algunas dependencias vienen de
  `packages.ever.co` (registro privado de Ever). Sin token → el build puede
  fallar al resolver esos paquetes. Es el riesgo #1 del rework.
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
| Sin `VERDACCIO_TOKEN` no build | Probar build local; si falla, ver qué package falta y decidir (sustituir por versión pública o pedir acceso). Documentar hallazgo. |
| Fork queda atrás de upstream | Flujo `estacion` + `git fetch upstream && git merge` programado; PRs propios. |
| Imágenes originales eternas en compose | Ramificar compose propio (`docker-compose.own.yml`) con GHCR tuyo. |
| Secrets en ambiente | `.env` nunca al repo; `GITHUB_TOKEN` con permiso `packages:write` (scoped). |

## 5. Próximo paso concreto (para CUANDO Samuel valide la demo)

1. `git fetch upstream` en el clon local → merge develop en rama `estacion`.
2. Probar **build local de la API** (medir tiempo/RAM) → documentar resultado.
3. Si el build local funciona → armar Actions en el fork (build-own.yml) →
   apuntar compose a GHCR de Samuel.
4. `DEMO=false` + seed limpio → primer tenant propio (Samuel) → mapear datos
   reales (flujos/mapeo-gauzy.md).

> No ejecutar aún: pendiente de exploración/validación de Samuel (está viendo
> la UI). Este doc queda como el plan de ejecución.