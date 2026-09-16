# Mapeo F3: pipeline freelance actual → entidades gauzy

> Verificado 16-sep-2026 sobre datos reales (`contactos.json`, `estado.json`)
> y API gauzy (demo). El pipeline liviano sigue siendo la fuente de verdad
> hasta que gauzy lo reemplace.

## 1. Datos reales (estado del pipeline)

| Dato | Valor actual | Fuente |
|---|---|---|
| Total vacantes procesadas | 82 | `propuestas-en-colas/estado.json` |
| En cola (pendientes de envío) | 33 | idem |
| Descartadas por requisito | 43 | idem |
| Enviadas | 3 (GiveDirectly · iMerit · Coalition) | idem |
| Borrador punte | 1 | idem |
| Contactos empresas (fichas) | 3 (conecty, viable-medical, imerit) | `contactos.json` |
| Fichas extendidas | conecty · viable-medical · plantilla | `fichas/*.md` |

### Contactos activos
- **Conecty** (SEGUIMIENTO): propuesta de automatización eSIM; esperando;
  borrador follow-up al viernes 18-sep si no responde. Sin email conocido.
- **Viable Medical** (CERRADA ❌): respuesta 14-sep "puesto ocupado".
  email `admin@viablems.com`.
- **iMerit** (SILENCIO ⏳): carta 13-sep, sin respuesta; re-recordatorio fin de
  semana. email `info@imerit.net`.

## 2. Mapeo de entidades

### 2.1 Empresas/contactos → `organization-contact` (gauzy)
| Real | gauzy | Ejemplo |
|---|---|---|
| ficha de empresa (conecty/imerit/viable) | `organization-contact` con contactType | `CLIENT` = posible cliente freelance; `LEAD` = candidatura en curso |
| relación empresa→proyecto | `contact` + `organization` vínculos | |
| estado del seguimiento (esperando/⏳/❌) | campo `notes` + `tags` | conecty → tag `seguimiento` |

> Propuesta de tipología: **CLIENT** para quien ya tiene un encargo/propuesta
> (conecty), **LEAD** para candidatura en curso (imerit, la cola), archive para
> cerradas (viable-medical).

### 2.2 Vacantes → `proposal` + `pipeline` + `candidate`
| estado.json | gauzy | Acción |
|---|---|---|
| `cola` | pipeline stage "Cola" | cada vacante = `candidate`/`proposal` en stage Cola |
| `enviada (auto) fecha → email` | pipeline stage "Enviada" + `email-history` | fecha + remitente como meta |
| `respondida fecha: asunto` | stage "Respondida" + nota + `email-history` | asunto en comentario |
| `descartada-requisito: …` | stage "Descartada" | motivo en comentario |

### 2.3 Correo → `email-history`
| Real | gauzy |
|---|---|
| `envios.log` | `email-history` (To, Subject, fecha) |
| bandeja `~/Trabajo/entrada/*/correo.txt` | `email-history` (Recipient/From, body) |
| `flujo0` informes | reportes gauzy / notes del contacto |

## 3. Scripts/macros → módulos
- Los scripts `griezz-*` se mantienen como CLI. gauzy los **invoca** por
  callout (webhook local) en vez de reescribirlos.
- La automatización diaria (props-dia 08:30 → autoprops 09:15) queda igual;
  gauzy solo refleja el estado (lectura de `estado.json` o vía script bridge).

## 4. Plan de migración (cuando el fork esté operativo)
1. Bridge local `gauzy-sync` (Python): lee `contactos.json` + `estado.json` →
   crea/actualiza `organization-contact` + pipeline stages en gauzy API
   (idempotente por id externo).
2. Mapear historial: emails enviados/recibidos → `email-history` por contacto.
3. Rutina: `griezz-props-dia` → después sincroniza a gauzy (mismo timer).
4. La UI gauzy queda como tablero visual; el liviano como respaldo y fuente
   para scripts hasta F6.

> Skills del lado gauzy: 148 módulos; los que importan primero:
> organization-contact, pipeline, proposal, candidate, email-history, tasks,
> goals. El resto se desactiva (feature toggle) para no abrumar la UI.