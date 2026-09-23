# Seguridad de datos — GitHub como base + sistema de mirrors (GRIEZZ)

Manual operativo del sistema de seguridad de SM_GRIEZZ (creada 23-sep-2026).
Principio: **la seguridad de los datos es lo primero**; todo repositorio tiene
copia en otro sitio desde el día 1.

## Arquitectura (5 capas)

| Capa | Qué | Dónde | Estado |
|---|---|---|---|
| 1. GitHub | Fuente de verdad, versionada con historial | Nube (cuenta `SamuelEcheverriUribe`) | ✅ activa |
| 2. Mirror local LUKS | 9 repos en bare mirror (todo historial+ramas+tags) | `/mnt/safe/github-backups/` (disco cifrado) | ✅ activa |
| 3. restic cifrado | Snapshot diario cifrado de Trabajo/Vida/Proyectos/repos/memoria/`.env`/motor | `/mnt/safe/backups/` (repo restic) | ✅ activa |
| 4. Segundo host nube | Mirror a GitLab/Gitea (otra nube) | Pendiente de cuenta | ⏳ diseñada |
| 5. Otro PC (futuro) | Bare mirror o restic en otra máquina → 3 sitios físicos | Pendiente de hardware | ⏳ diseñada |

## Operación manual

```bash
# Mirror de GitHub → disco seguro (crea o actualiza los 9 repos)
~/.local/bin/griezz-github-mirror.sh
# Log: ~/.local/share/griezz/github-mirror.log

# Backup restic (manual, mismo repo que el timer)
systemctl --user start griezz-restic-backup.service
# Log: ~/.local/share/griezz/restic-backup.log

# Ver estado de los timers
systemctl --user list-timers griezz-github-mirror.timer griezz-restic-backup.timer

# Listar snapshots de restic
restic -r /mnt/safe/backups snapshots --password-file=$HOME/.config/restic/restic-pass

# Verificar que un repositorio mirror tiene las ramas esperadas
git -C /mnt/safe/github-backups/<repo>.git branch -a
git -C /mnt/safe/github-backups/<repo>.git log --oneline -1
```

## Automatización (timers)

- **`griezz-github-mirror.timer`**: 06:15 diario, `Persistent=true` (corre al
  encender si la hora pasó apagado).
- **`griezz-restic-backup.timer`**: 04:00 diario (pre-existente), retención
  7 diarios / 4 semanales.

Ambos requieren que `/mnt/safe` (disco LUKS) esté montado; si no lo está, el
script registra error y sale (no rompen nada).

## Reglas de administración (críticas)

1. **La pass de restic NUNCA va en el script** — vive en
   `~/.config/restic/restic-pass` con permisos 600 (regla IRROMPIBLE 17-sep).
   Verificado: script actual la lee vía `--password-file`.
2. **Nada de secretos (`.env`, keys) en GitHub** — `.env` solo existe local y
   entra a restic cifrado.
3. **Datos de clientes (Suiza/HITL) nunca a nube pública** — la capa 2/3 local
   cifrada es su residencia.
4. Si se crea un repo nuevo en GitHub → añadirlo a la lista `REPOS=` del script
   `griezz-github-mirror.sh` (y a este inventario).
5. Si `/mnt/safe` está desmontado >1 día: correr los scripts manualmente tras
   montarlo (no hay retención de deuda, se pierde el día).

## Encriptación del disco seguro

- `/mnt/safe` = LUKS (un solo dispositivo, cipher AES). Montaje bajo demanda;
  nunca montado en reposo. Los backups viven dentro de esa capa cifrada.

## Para montar la capa 4 (segundo host, cuando haya cuenta)

1. Crear cuenta/proyectos espejo en GitLab (o Gitea autoalojado / Codeberg).
2. Por cada repo: `git push --mirror <url-gitlab>` desde el bare local:
   ```bash
   git -C /mnt/safe/github-backups/<repo>.git push --mirror https://gitlab.com/USER/<repo>.git
   ```
3. Opcional: añadir al script una fase de push a GitLab con la misma vara.
   Recomendación: que la capa 4 corra SOLO cuando la capa 2 ya subió el mirror
   local (no depender de red directa a GitHub).