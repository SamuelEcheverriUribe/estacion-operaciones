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

## Para montar la capa 4 (segundo host: Codeberg, recomendado)

La página real de GitLab.com pide verificación con tarjeta y bloqueó la cuenta
Samuel (23-sep) por su filtro anti-abuso de cuentas nuevas — descartado en la
práctica para el mirror. **Codeberg** (Forgejo, Europa) es la vía sin fricción:
sin tarjeta, sin bloqueos así.

1. **Crear cuenta** en https://codeberg.org (sin tarjeta).
2. **Generar token**: Codeberg → Settings → Applications → Generate Token,
   scopes `read:user` + `write:repository`.
3. **Rellenar** `~/.config/griezz/mirror-host2.env` (permisos 600, formato
   ya creado): `HOST2_URL="https://codeberg.org"`, `HOST2_USER="<tu usuario>"`,
   `HOST2_TOKEN="<token>"`, `HOST2_VISIBLE="false"` (repo privados).
4. **Correr** el script: el mirror local (capa 2) ya se hizo, ahora la fase 2
   crea los repos en Codeberg vía API y hace `git push --mirror` de los 9.
   ```bash
   ~/.local/bin/griezz-github-mirror.sh
   tail -20 ~/.local/share/griezz/github-mirror.log
   ```
5. El timer diario (06:15) ya ejecuta fase 1 + fase 2 automáticamente.

> Alternativas equivalentes si Codeberg no convence: Gitea/Forgejo autoalojado
> (futuro servidor personal) o una instancia GitLab self-hosted. El mecanismo
> es el mismo: `git push --mirror` desde el bare local.

## Para montar la capa 5 (otro PC / disco físico)

Cuando exista portátil u otro disco: duplicar `/mnt/safe/github-backups/`
con el mismo script o un rsync, o añadir el segundo PC como destino restic
(`restic -r ssh://...`). Objetivo: copia en 3 sitios físicos distintos.