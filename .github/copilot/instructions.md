# Instrucciones de GitHub Copilot — Estación GRIEZZ

> Este archivo condiciona el comportamiento de Copilot dentro de este
> repositorio y los conectados (automatizaciones, freelance-pipeline, ghostink).
> Copilot lo lee de `AGENTS.md` en repos de su agente o de instrucciones de
> organización/cuenta cuando está configurado.

## Quién soy
- Samuel Echeverri Uribe (SM_GRIEZZ), freelancer: automatización de datos,
  IA-training, desarrollo Godot/GDScript, scripts Python/Node, pipeline freelance.
- Idioma de trabajo: **español** para explicaciones; **código/comentarios en
  inglés** salvo que se pida lo contrario.
- Ecosistema opensource/coste-0. Nunca proponer herramientas propietarias de
  pago sin preguntar.

## Reglas de seguridad (obligatorio, no salvo excepción)
1. NUNCA escribir claves, tokens o secretos en código, logs o commits. Nunca
   "placeholder tipo dame_tu_key". Validar con `griezz-check` donde exista.
2. Nunca `curl | bash` ni scripts no revisados.
3. Datos de clientes (HITL/Suiza) nunca al free-tier de modelos que entrenen con
   ellos; dentro de Copilot recordarlo como contexto.
4. Preferir leer credenciales de entorno (env vars o recomendaciones del
   lanzador `griezz`), nunca hardcodear.

## Estilo de código
- Python: PEP8, type hints cuando aporten, funciones cortas con propósito único.
- GDScript: convenciones de Godot 4 (snake_case, @export, señales).
- JavaScript/TS: formato Prettier, semicolon, arrow functions.
- Validar antes de prometer: si no sabés de un resultado, decirlo (verificado /
  probable / incierto).

## Contexto de trabajo del repo
- `automatizaciones/`: limpieza CSV/Excel, extractor de facturas, vigilante de
  precios, OCR, bandeja de clientes, análisis web, orquestador por horario y CI.
- `freelance-pipeline/`: escaneo diario de vacantes (RemoteOK/Remotive/Jobicy +
  Computrabajo), criterios en cascada, GitHub Actions (cron 08:30/15:00 COT),
  borradores vía `griezz-correo` → webmail Infomaniak.
- `ghostink/`: APK Android Kotlin/Compose para notas+math (MyScript iink), GPL-3.
- Godot: `Laboratiorio Godot` (GDScript, 2D), motor 4.7.2.

## Ante dudas pedir lo mínimo
- Si falta contexto del usuario (decisiones de negocio), preguntar en 1 línea.
- Si una tarea es ambigua: listar 2 rutas válidas en vez de preguntar de más.