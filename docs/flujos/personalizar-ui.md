# Guía: personalizar gauzy a tu gusto (UI)

> Para Samuel, explorando la demo en http://localhost:4200 (admin@ever.co/admin).
> Lo que se ve abajo **puede cambiarse sin tocar código** (menú lateral de
> configuración) y lo demás requiere rework del fork.

## 1. Lo que cambias SIN tocar código (menú Configuración)

- **Tema de apariencia** (icono de paleta, barra izquierda):
  - `default` (claro), `dark` (oscuro), `cosmic` (púrpura/espacial), `corporate`,
    + selector de **color de acento** (sidebar de tema: usuarios/estados se colorean).
  - Código base: `packages/ui-core/theme/src/lib/themes/{default,dark,cosmic,corporate}.ts`.
- **Idioma**: menú de idioma (derecha superior) → gauzy está en 10+ idiomas
  (Crowdin). Es: ¿"es"? (check en demo).
- **Moneda/ubicación/zonas**: defaults en `.env`/config (`DEFAULT_CURRENCY`,
  `DEFAULT_LATITUDE/LONGITUDE`).
- **Cuenta y tenant**: datos de la organización desde el dashboard (nombre,
  logo, email de contacto), roles empleados.
- **Features toggle**: módulos enteros se encienden/apagan (organizar la UI:
  dejar solo lo que usas = real estate limpio).

## 2. Lo que cambias desde .env (al montar tu fork)

| Variable | Efecto |
|---|---|
| `APP_NAME`, `APP_LOGO`, `APP_SIGNATURE` | marca = tu nombre (GRIEZZ / freelance) |
| `APP_LINK` | URL de login |
| `DEFAULT_CURRENCY` | moneda base de reportes |
| `DEFAULT_LATITUDE/LONGITUDE` | ubicación del dashboard |
| `DEMO=true/false` | datos semilla / instalación limpia |
| `GOOGLE_MAPS_API_KEY` | mapas (opcional) |
| `SENTRY_*`, `POSTHOG_*` | telemetría (dejar apagada: privacidad) |

## 3. Lo que requiere rework del fork

- **Vista "Estación freelance"** (tuya): página que combine mis pendientes,
  pipeline de candidaturas y seguimiento de contactos en una pantalla.
- **Logo/marca personal** (si no alcanza con APP_LOGO).
- **Módulo de scripts/macros**: panel que liste `griezz-*` y lance botones
  (execute local → API call). Es lo que Samuel pidió en la estación.
- **Conexión correo**: vistas de email-history por contacto.

## 4. Qué dejar / qué apagar (sugerencia inicial)

**Dejar**: Dashboard, Contacts (CRM), Sales Pipelines/Proposals, Email History,
Tasks, Goals, Organization settings.

**Apagar por feature-toggle** (si la UI te satura): Payroll, Time Tracking/
Timesheets, Inventory, Equipment, Reporting avanzado, ATS entrevistas, Alerts
antics, Jira/GitHub (si no los usas aún).

## 5. Mirada honesta sobre el demo

- El demo prebuilt trae **146 contactos semilla falsos** y mucha funcionalidad
  encendida: es solo para explorar. Al montar TU fork pondrás `DEMO=false` y
  carpetas limpias + tu tenant.
- Password demo `admin` y todo en local: jugar sin miedo (es tu máquina, no
  expones nada). Para resetear: `docker compose down -v` y subir de nuevo.

---
*Fuente: código `packages/ui-core/theme`, `docker-compose.demo.yml`, config env.*