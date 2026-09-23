# ESTACIÓN DE OPERACIONES GRIEZZ

Sistema operativo de trabajo del freelancer Samuel Echeverri Uribe (SM_GRIEZZ):
una estación que unifica contactos, pipeline de propuestas, correo, scripts/macros
y automatizaciones; desplegada con contenedores, alojada en GitHub y conectable
desde una app móvil por cuenta.

> Regla de construcción: **documentación-primero**. Cada flujo se escribe y se
> entiende antes de implementarse. Lo escrito sirve de base para montar sistemas
> similares (potencialmente para clientes como Conecty) y para aprender el flujo
> de servicios completo.

---

## 1. Visión

- **Hoy**: el trabajo freelance vive en `~/Trabajo/Freelance` (contactos.json,
  fichas, propuestas-en-colas, envios.log) + scripts `~/.local/bin` + suite
  `~/Proyectos/automatizaciones` (flujo0, analiza-web, vigila-ofertas, etc.).
  Funciona, pero está disperso en archivos planos y depende de esta sola máquina.
- **Meta**: convertirlo en un **sistema con API y cuentas**, desplegado con
  contenedores, con el repositorio en GitHub como base de trabajo remoto, y un
  cliente móvil que se conecta por cuenta.
- **Aprendizaje**: construirlo es aprender el flujo de servicios moderno
  (Git + contenedores + API + auth + móvil). Esa habilidad es exactamente la que
  Conecty (o cualquier cliente de automatización) pagaría.

## 2. Decisiones de base (registradas)

- **Plataforma**: `ever-co/ever-gauzy` adoptada de lleno como estación de
  trabajo (decisión de Samuel, 16-sep). Es ERP/CRM/HRM/ATS opensource
  (Angular + NestJS, 148 módulos, MCP server). Se usará de base y se hará
  rework/adaptación a las necesidades reales.
- **GitHub**: cuenta `SamuelEcheverriUribe` (token `repo`+`workflow`, gh
  autenticado). El código vivirá en forks/repos propios.
- **Contenedores**: Docker (autorizado instalarlo con sudo, 16-sep). Podman ya
  está instalado como alternativa sin sudo.
- **App móvil**: se construirá sobre la misma API; conexión por cuentas (JWT).
- **Fuente de verdad mientras exista duplicación**: el pipeline liviano de
  `~/Trabajo/Freelance`. La estación gauzy la consume; no se elimina hasta que
  gauzy la reemplace por completo.

## 3. Arquitectura objetivo

```
                      ┌──────────────────────────────────────────────┐
                      │             ESTACIÓN OPERACIONES              │
                      │                                              │
   móvil (cuenta) ──► │  API gauzy (NestJS, REST+GraphQL, JWT auth)  │
                      │        ▲            ▲            ▲           │
                      │        │            │            │           │
                      │   CRM/contactos │  pipeline/ATS │  correo    │
                      │   propuestas   │  fichas       │  flujos    │
                      │        │            │            │           │
                      │  ┌──────────┴────────┴────────/  scripts &  │
                      │  │  módulos de integración        macros     │
                      │  └──────────┬──────────┬───────────/         │
                      │             ▼          ▼                     │
                      │        Postgres/    almacén local            │
                      │        SQLite       (scripts, plantillas)    │
                      └──────────────────────────────────────────────┘
                              ▲                    ▲
                              │                    │
                     GitHub (remoto)      móvil (REDMI, futuro)
                     base de trabajo     acceso por cuenta
                     + CI/CD
```

Capas:
1. **GitHub**: repositorio (fork de gauzy + repo de configuración/operación),
   trabajo remoto seguro, CI.
2. **Contenedores**: Docker Compose levanta API + web + DB (dev en SQLite,
   prod en Postgres). Volúmenes para datos persistentes; redes internas.
3. **API con cuentas**: gauzy trae auth (tenant, roles, JWT). Los datos del
   freelancer (contactos, pipeline, correo) se exponen por API.
4. **App móvil**: cliente que entra por cuenta a la API (lo que se aprende acá
   es exactamente el contrato API -> cliente que se reusa en el móvil).
5. **scripts/macros/flujos de correo**: conectados a la estación como módulos
   (patrón plugin gauzy o ejecución local invocada por la API).

## 4. Mapa de flujos (docs/flujos/)

| Flujo | Qué documenta | Archivo | Estado |
|---|---|---|---|
| Pipeline de propuestas | cola → carta → envío → respuesta | `flujos/pipeline-props.md` | ⏳ |
| Correo (flujo0) | bandeja IMAP → informe → respuesta | `flujos/correo-flujo0.md` | ⏳ |
| Contactos y fichas | perfil por contacto/empresa | `flujos/contactos-fichas.md` | ⏳ |
| Scripts y macros | inventario de automatizaciones | `flujos/scripts-macros.md` | ⏳ |
| Móvil + cuentas | cómo entra el cliente móvil | `flujos/movil-cuentas.md` | ⏳ |

## 5. Plan de fases

- **F1 · Fundación docs y repo** (en curso): este documento + inventario de
  flujos actuales + git config + repo GitHub.
- **F2 · Contenedores**: instalar Docker, compose levantar gauzy (dev SQLite),
  aprender volúmenes/redes/config.
- **F3 · Mapeo de datos**: migrar/mapear contactos, pipeline, fichas y correo a
  entidades gauzy (contacts, pipeline/proposals, email history).
- **F4 · Integraciones**: conectar scripts/macros/correo actuales como módulos
  de la estación (plugin gauzy o bridge local).
- **F5 · API + cuentas**: exponer servicios por API JWT; cliente de prueba
  (curls/script) que consume la estación remotamente.
- **F6 · App móvil**: cliente móvil que entra por cuenta (primero versión web
  móvil, luego app nativa si el interés lo justifica).

## 5b. GitHub como docker personal + IA de GitHub (nueva, 23-sep)

- **Entorno reproducible versionado**: `.devcontainer/` (Node 22 + Python 3.12 +
  Docker-in-Docker + VS Code con Copilot/Python/GitLens/Actions) + `.vscode/`
  (extensiones y settings) + `.github/copilot/instructions.md` (Copilot
  condicionado a la labor de Samuel: español/explicaciones, código en inglés,
  reglas de seguridad, ecosistema opensource, estilo por lenguaje, contexto de
  repos automatizaciones/freelance-pipeline/ghostink/Godot).
- **Precedente USB**: este entorno es lo que viaja en la estación USB. Cualquier
  PC: clonar → `gh auth` → "Reopen in Container" → misma estación.
- **Estado honesto Copilot (verificado 23-sep)**: token gh sin scope `copilot`;
  `user/copilot` 404 → cuenta NO activa aún. Plan **Free** disponible; activar
  y queda afinado. Extensión `github.copilot-chat` ya instalada en VS Code.
- Cómo usarlo en PC / tablet (Termux+code-server) / USB → `usos/estacion-editor.md`.
  Checklist de activación ahí.

## 5c. Godot en la tablet + repos + plantillas (nueva, 23-sep)

- **Godot editor Android oficial**: existe (4.7.2 stable, APK en
  godot-builds/releases ~656MB). Lenovo Tab M11 (arm64) lo corre. Solo GDScript,
  sin exportar, render Compatibility para 2D. UI táctil NO optimizada → usar
  ratón+teclado.
- **Conexión a GitHub desde la tablet**: método A git+gh en Termux (flujo
  remoto real) o método B Syncthing (ya activo). Ambos en la guía.
- **Repo pieza-patrón "plantillas"**: se puede tener un repo template con
  esqueletos de proyectos Godot para clonar desde GitHub ("create from
  template") según el tipo de juego. Propuesta en `usos/godot-tablet-repos.md`.
- **Estado repos verificado 23-sep**: `game-profiles` (privado) está VACÍO en
  GitHub — el laboratorio Godot existe local y no está versionado aún.
- Guía completa → `usos/godot-tablet-repos.md`.

## 6. Acuerdos de operación

- Documentar antes de implementar (regla del usuario).
- El pipeline liviano sigue activo hasta que gauzy lo reemplace (no romper lo
  que ya produce dinero/avance).
- Datos de clientes (Suiza/HITL) nunca a la nube pública: esta estación es
  personal y local; GitHub solo para código (sin secretos: `.env` ignorados).
- App móvil de prueba con datos ficticios hasta validar.

## 7. Seguridad de datos — GitHub como base + mirror (nueva, 23-sep)

Regla nueva del usuario: **la seguridad de los datos es lo primero**. GitHub es
el servicio principal de guardado; todo repositorio tiene copia en otro sitio
(mirror) desde el día 1.

- **Capa 1 · GitHub (nube)**: fuente de verdad, versionada con historial.
- **Capa 2 · Mirror local LUKS**: `/mnt/safe/github-backups/` (disco USB cifrado
  con LUKS, 208GB libres) guarda los **9 repos en bare mirror** (`--mirror`: todo
  el historial + ramas + tags). Script `griezz-github-mirror.sh` (actualiza o
  crea), timer diario 06:15 (`griezz-github-mirror.timer`, con `Persistent=true`).
- **Capa 3 · restic cifrado diario**: `griezz-restic-backup.sh` (04:00) respalda
  `~/Trabajo`, `~/Vida`, `~/Proyectos`, `~/Documentos/GitHub`, la memoria GRIEZZ,
  `.env` y el motor al repositorio restic `/mnt/safe/backups` (pass en archivo
  600, nunca en el script — regla IRROMPIBLE). Retención: 7 diarios / 4 semanales.
- **Capa 4 · segundo host en la nube (diseñada, pendiente de cuenta)**: mirror a
  GitLab/Gitea cuando exista credencial → "otro sitio en la nube" del usuario.
  Mismo patrón `git push --mirror` que la capa 2, otro remoto.
- **Capa 5 · otro PC (futuro)**: cuando exista portátil/otra máquina, bare
  mirror local ahí (o restic a su disco) cierra la copia en 3 sitios físicos.

Inventario repos protegidos (verificado 23-sep): `estacion-operaciones`,
`godot-templates`, `freelance-pipeline`, `ghostink`, `vanilla-os-griezz`,
`game-profiles`, `dotfiles`, `ever-gauzy`, `automatizaciones`.
Detalle → `usos/seguridad-datos.md`.

---

*Creado 16-sep-2026 por GRIEZZ. Se amplía en docs/flujos/, docs/infra/ y usos/.*