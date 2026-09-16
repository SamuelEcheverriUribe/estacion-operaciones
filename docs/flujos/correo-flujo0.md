# Flujo: Correo (cuenta, bandeja, envío, respuestas)

> Cuenta real: Infomaniak (`samuel.echeverri@ikmail.com`). Credenciales en
> `flujo0/config.privado.json` (600, NO al repo, NO a la nube).

## 1. Componentes

- **IMAP** (descarga): flujo0 `descargar_imap()` — hosts IMAP4_SSL 993.
  - por defecto `UNSEEN`; flag `--sentsince N` rescata leídos recientes
    (dedup Message-ID en `bandeja/procesados.txt`).
  - al procesar marca `\Seen` (la bandeja del webmail queda leída).
- **SMTP** (envío): `griezz-correo` — puertos 465 SSL / 587 STARTTLS.
  - guarda en carpeta Enviados (IMAP) + `envios.log` + alerta al usuario.
- **Bandera de entrada**: `~/Trabajo/entrada/<id_contacto>/<fecha>/correo.txt`
  (+ adjuntos + `enlaces.txt` verificado por analiza-web).
- **Informe**: `generar_informe()` → `~/Trabajo/Freelance/informes/*.md`
  + envío local (KDE Connect móvil REDMI).

## 2. Ciclo de un correo entrante (flujo0 revisar)

1. Conectar IMAP (credenciales desde config[privado]).
2. Buscar mensajes (UNSEEN o SENTSINCE).
3. Dedup por Message-ID (procesados.txt) para no reprocesar.
4. Para cada mensaje nuevo:
   - extraer `From` → `buscar_contacto(de, contactos.json)` → ficha o
     `sin_clasificar`.
   - guardar `correo.txt` + adjuntos + enlaces verificados.
   - generar informe MD y enviarlo (PC + REDMI).
5. `marcar_respuestas_pipeline()`: si el remitente coincide con un email de
   `contactos-vacantes.json` → esa vacante pasa a
   `respondida <fecha>: <asunto> (de <email>)` en `estado.json` y alerta.
6. `comando_listar` muestra el estado de la bandeja.

## 3. Ciclo de envío (sepa: props-dia → autoprops → griezz-correo)

```
griezz-props-dia (08:30, timer)        paquete del día + cartas b-XX.txt
   └─> estado.json: vacantes en "cola"
griezz-autoprops (09:15, timer, --max 6)
   ├─ por cada vacante en cola CON email:
   │     cuerpo b-XX.txt + asunto "Application — <título>" + CV adjunto
   │     → griezz-correo (SMTP) → Enviados + envios.log + alerta
   │     → estado: "enviada (auto) <fecha> → <email>"
   └─ sin email → se queda al portal (no se inventa destino)
```

## 4. Respuestas (lección aprendida 16-sep)

- Caso real: Viable Medical respondió "puesto ocupado"; llegó leído al webmail
  y `UNSEEN` no lo veía → se perdió en el flujo.
- Solución (implementada): `revisar --sentsince N` rescata leídos recientes.
- Reglas para el futuro:
  - respuestas de vacantes → `contactos-vacantes.json` (dominio) marca estado.
  - respuestas de contactos (conecty/imerit/viable-medical) → `contactos.json`
    (caso a caso; hoy se marca manual).
  - el primero que responde guía la atención del día (alerta "pregunta").

## 5. Mapeo a gauzy

| Actual | gauzy |
|---|---|
| correo.txt / informes/*.md | `email-history` (correo por contacto/propuesta) |
| envios.log | `email-history` (bandeja enviados) |
| contactos.json / sin_clasificar | `contact` + `organization-contact` / inbox |
| estado.json (respondida) | campo estado en `proposal` |

## Notas

- Secretos solo en `flujo0/config.privado.json` (modo 600) y se exportan en el
  lanzador. Nunca en comandos de terminal (historial) ni en el repo.
- El flujo por GitHub: solo código plantilla + docs; las credenciales y datos de
  cliente viven en el PC (tenant local).