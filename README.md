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

## 6. Acuerdos de operación

- Documentar antes de implementar (regla del usuario).
- El pipeline liviano sigue activo hasta que gauzy lo reemplace (no romper lo
  que ya produce dinero/avance).
- Datos de clientes (Suiza/HITL) nunca a la nube pública: esta estación es
  personal y local; GitHub solo para código (sin secretos: `.env` ignorados).
- App móvil de prueba con datos ficticios hasta validar.

---

*Creado 16-sep-2026 por GRIEZZ. Se amplía en docs/flujos/ e docs/infra/.*