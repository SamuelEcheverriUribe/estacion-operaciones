# Flujo: Scripts y macros — inventario (mapeo a gauzy)

> Fuente: `~/.local/bin/` (scripts con `griezz-*`) + `~/Proyectos/automatizaciones/`
> (suite Python por nicho). Verificado 16-sep-2026.
>
> Objetivo en la estación: cada item pasa a ser un **módulo invocable** (patrón
> plugin gauzy o bridge local llamado por la API), con su propia ficha.

## A. Scripts personales del sistema (griezz-*)

| Script | Función | Prioridad gauzy |
|---|---|---|
| `griezz` | lanzador del Orquestador (motor GRIEZZ) | — (sigue igual) |
| `griezz-alert` | notificación + sonido al terminar/esperar | auxiliar (se reutiliza) |
| `griezz-autoprops` | envío automático de propuestas (cola → correo, tope 6/día) | **core pipeline** |
| `griezz-contacto` | extraer emails de la cola de vacantes | **core pipeline** |
| `griezz-correo` | enviar/draft con cuenta Infomaniak | **core correo** |
| `griezz-props-dia` | genera el paquete diario de propuestas (08:30) | **core pipeline** |
| `griezz-reporte-dia` | parte diario consolidado (05:30) | informes |
| `griezz-jobs` | scraping de vacantes (fuentes) | **core pipeline** |
| `griezz-pendientes` | pendientes del sistema residente | vitrina |
| `griezz-guia-dia.sh` | guía diaria (04:55) | informes |
| `griezz-revision-semanal.sh` | consolidación semanal (30 min) | informes |
| `griezz-restic-backup.sh` | respaldo restic | infra/servicio |
| `griezz-health` / `griezz-salud` | watchdog de salud del sistema | infra/servicio |
| `griezz-update.sh` | auto-update GRIEZZ | infra/servicio |
| `griezz-backup` / `griezz-restore` | respaldo/restauración de config | infra/servicio |
| `griezz-entorno` / `griezz-mira` / `griezz-ver` | operar ventanas / leer pantalla | capacidades ARES |
| `griezz-hub` / `griezz-idea` / `griezz-ideas-triaje.sh` | bandeja de ideas + triaje | sistema residente |
| `griezz-wallpaper` / `griezz-voz` / `griezz-mic.py` | configuración visual/voz | auxiliar |

## B. Suite de automatizaciones (Python, `~/Proyectos/automatizaciones/`)

| Módulo | Función | Relación |
|---|---|---|
| `flujo0` | bandeja IMAP → ficha contacto → informe MD → teléfono | **core correo** |
| `analiza-web` | verificar enlaces (fetch seguro, sin JS) | guardián |
| `vigila-ofertas` | vigilar ofertas por nicho (e.g. Conecty eSIM) | **core pipeline** |
| `extractor-facturas` | extraer datos de facturas | datos |
| `limpiadatos` | limpieza de CSV/Excel | datos |
| `observa-precio` | monitoreo de precios | datos |
| `scanner-docs` | OCR/reconocimiento de documentos | datos |
| `orquesta` | orquestador por horario (timers) | infra |

## C. Datos operativos (fuente de verdad actual) — a mapear

| Ruta | Contenido | Entidad gauzy objetivo |
|---|---|---|
| `~/Trabajo/Freelance/contactos.json` | fichas de contacto (contacto, proyecto, correos) | `contact` / `organization-contact` |
| `~/Trabajo/Freelance/fichas/*.md` | ficha extendida por empresa | `contact` (notas) |
| `~/Trabajo/Freelance/propuestas-en-colas/estado.json` | estado por vacante (cola/enviada/respondida…) | `proposal` / `pipeline` |
| `~/Trabajo/Freelance/propuestas-en-colas/b-XX.txt` | cartas de propuesta | `proposal` (adjunto) |
| `~/Trabajo/Freelance/propuestas-en-colas/contactos-vacantes.json` | email por vacante | `proposal` + `contact` |
| `~/Trabajo/Freelance/propuestas-en-colas/envios.log` | log de envíos | `email-history` |
| `~/Trabajo/entrada/**` | bandeja flujo0 (correo.txt, adjuntos) | `email-history` |
| `~/Vida/ideas/entrada/` | bandeja de ideas | `goal`/idea |

## D. Clasificación para la estación

1. **Core de valor** (primeros en integrar): `correo`, `pipeline` (props-dia,
   autoprops, contacto, jobs), informes (reporte-dia).
2. **Servicios/infra** (arrancan con la estación): backups, health, update.
3. **Capacidades ARES** (se mantienen como CLI injertable): entorno, ver, voz.
4. **Datos/auxiliares**: limpiadatos, facturas, scanner → se exponen como
   funciones de la API cuando haga falta.

## E. Decisiones

- No se migra nada de código a gauzy todavía: primero se integra gauzy como
  contenedor (F2), se mapean datos (F3) y se conecta como módulos (F4).
- Los scripts `griezz-*` siguen siendo la implementación; gauzy los **invoca**
  (callout) o los reemplaza solo cuando hay equivalencia real en el módulo.
- Backups y secretos: fuera del repo (`.env` ignorado), datos del freelancer
  como tenant aislado.