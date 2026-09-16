# Flujo: Pipeline de propuestas (cola → carta → envío → respuesta)

## 1. Origen de la cola

- `griezz-jobs` (fuentes): Computrabajo CO (portales), RemoteOK, Jobicy, otros.
  Filtrar por nicho (data entry / asistente virtual / automatización).
- Criterio mínimo (regla):
  1. No pedir título universitario (descartar si exige "universidad").
  2. Contrato/condiciones razonables y canal accesible.
  3. Portal que cobre → descartado.

## 2. Del feed al paquete del día (`griezz-props-dia`, timer 08:30)

```
jobs/ (vacantes crudas del scraper)
  └─> filtro (nicho + requisitos) → candidatas con b-XX.txt (carta adaptada)
  └─> estado.json: "cola"
  └─> Propuestas-dia-YYYY-MM-DD.md  ← paquete visible aprobado
```

Estructura del pack (formato leído por `griezz-autoprops`):

```
## N. <título> — <empresa> (<fuente>) · ...
- [Ver](<url de la vacante>)
- **Qué piden**: <descripción>...
- **Propuesta lista**: `b-XX-<slug>.txt`
- **Destino**: <canal>
- **¿Enviar?** [ ] SÍ  [ ] No  (decisión de Samuel)
```

## 3. Carta de propuesta (b-XX.txt)

Cuerpo estándar (adaptado por vacante): presentación, motivación específica del
puesto, skills (datos/automatización), remoto + disponibilidad inmediata, cierre
con firma. Adjunta `CV_Samuel_DataEntry_EN.pdf`.

## 4. Contacto del destino (`griezz-contacto`)

- Extrae emails reales de cada vacante en cola (fetch público, filtro
  NO_REPLY/REDES/PORTALES) → `contactos-vacantes.json`.
- Portal (Computrabajo/RemoteOK) normalmente NO expone email → "portal/manual".
- Jobicy/otros a veces sí (ej: `careers@givedirectly.org`).
- Regla de calidad: **sin email real, no se envía directo** (queda al portal).

## 5. Envío automático (`griezz-autoprops`, timer 09:15, --max 6)

- Solo vacantes en estado **`cola`** con **email** en `contactos-vacantes.json`.
- asunto: `Application — <título>` · cuerpo: b-XX.txt · adjunto: CV EN.
- 1 envío por corrida (retorno para acotar riesgo) hasta `--max`.
- Estado → `enviada (auto) <fecha> → <email>`.
- Alertas: `griezz-alert` al usuario.

## 6. Estado del ciclo (estado.json)

```
"cola"                     → pendiente de envío
"enviada (auto) fecha → e" → enviada por la estación
"enviada fecha → e"        → enviada manualmente
"respondida fecha: asunto (de email)"  → hubo respuesta (flujo0 la marca)
"descartada-requisito: …"  → no cumple criterio
```

## 7. Reglas vigentes (decisión de Samuel)

- Doc primero, implementar después.
- Tope diario: 6 envíos automáticos/día (controla reputación).
- Solo dominio válido (anti-spam); nada sin email real.
- Respuestas se atienden el mismo día (alerta "pregunta").

## 8. Mapeo a gauzy

| Actual | gauzy |
|---|---|
| jobs/ + filtro | `pipeline` (sales pipeline de candidatura) |
| Propuestas-dia / b-XX | `proposal` + adjuntos |
| estado.json | estado `proposal` (cola/enviada/respondida) |
| contactos-vacantes.json | relación propuesta→contact |
| informes | `email-history` |