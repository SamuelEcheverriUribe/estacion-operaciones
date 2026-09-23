# ESTACIÓN EDITOR — GitHub como "docker personal" + IA de GitHub

> Precedente para la USB de trabajo: TODO el entorno de edición queda definido
> como código en este repo (devcontainer + .vscode + instrucciones Copilot),
> conectado a GitHub. Cualquier máquina reproduce la estación con `gh` +
> VS Code + hacer clic en "Reopen in Container".

## Qué es esto (la idea completa)
- **GitHub = docker personal**: no "docker container" de la PC, sino el patrón de
  que el ENTORNO se versiona y se sube a GitHub (devcontainer + config + instrucciones).
- **Precedente USB**: lo que quede aquí se copia a la USB de estación → el mismo
  entorno en cualquier PC. Primero el código / último la máquina.

## Capas
| Capa | Archivo | Qué hace |
|---|---|---|
| Entorno reproducible | `.devcontainer/devcontainer.json` | Levanta Node 22 + Python 3.12 + Docker-in-Docker + VS Code con extensiones (Copilot, Python, GitLens, Actions) |
| IA de GitHub condicionada | `.github/copilot/instructions.md` | Le dice a Copilot: español/explicaciones, código en inglés, reglas de seguridad (nunca claves), ecosistema opensource, estilo por lenguaje, contexto de tus repos |
| Editor automático | `.vscode/extensions.json`, `settings.json` | Cuando abres el repo en VS Code, sugiere/instala las extensiones exactas y la config (formato, lint, git) |
| Uso práctico | `usos/estacion-editor.md` (este) | Cómo usarlo en PC, tablet (Termux+code-server), y qué límites tiene Copilot hoy |

## Cómo usarlo

### En el PC (donde tienes VS Code con copilot-chat ya instalado)
1. Abrir repo con VS Code: `code ~/Proyectos/estacion-operaciones`
2. VS Code detecta `.devcontainer` → "Reopen in Container" para entorno completo.
   (O trabaja local: la config `.vscode` aplica igual.)
3. Git ya está autenticado con `gh` → clonar/pull/push sin fricción.
4. Copilot: instalar la extensión `GitHub Copilot` y firmar con tu cuenta GitHub.
   - Chat: **Copilot Chat** (sí tienes la extensión) — pregunta sobre el código.
   - Completar código: pendiente de que la cuenta active Copilot (ver límites).

### En la tablet (Termux + code-server) — "editar y revisar desde el móvil"
1. Termux: `pkg install tur-repo && pkg install code-server` (vía TUR).
2. `code-server` → abre `http://127.0.0.1:8080` en el navegador.
3. Abres el repo clonado (git + gh) y editas → push a GitHub.
4. Copilot en code-server: instalar extensión desde marketplace. En Android, la
   extensión Copilot tiene soporte limitado (web extension); si falla, editar +
   push normal funciona igual (la IA completa en PC/USB).

### En cualquier PC ajena (patrón USB / estación)
1. Arrancar el USB (ver `~/Trabajo/Estacion-USB/checklist-armado.md`).
2. `git clone` de `estacion-operaciones` + `hub` de automatizaciones/pipeline.
3. `gh auth login` (tu cuenta) → VS Code abre repo → "Reopen in Container" → la
   estación queda igual en cualquier hardware.

## Límites honestos HOY (verificado 23-sep)
- Tu token `gh` NO tiene el scope `copilot`, y `gh api user/copilot` responde 404
  → **Copilot NO está activo en tu cuenta todavía**.
- La extensión `github.copilot-chat` SÍ está instalada en tu VS Code.
- **Copilot tiene plan Free** (completions y chat con topes mensuales). Activar:
  - En VS Code: extensión "GitHub Copilot" → Sign in → aceptar el plan gratuit.
  - O en github.com → Settings → Copilot → habilitar plan Free.
- La config (instructions + extensiones + .vscode) queda lista y condicionada:
  cuando actives Copilot, ya viene afinado a tu labor. No se pierde nada.
- GDScript/Godot: el editor GDScript de VS Code funciona; Godot mismo (rendering)
  corre local o en la USB, NO dentro del devcontainer (no hay GPU en contenedor).

## Checklist de activación (pendiente de S.M.)
- [ ] Activar Copilot Free en github.com → Settings → Copilot.
- [ ] En VS Code instalada extensión Copilot → Sign in (la de chat ya está).
- [ ] Probar en `estacion-operaciones`: abrir `docs/`, preguntar a Copilot Chat
      "resumí el estado de la estación según el README" → debe responder en español.
- [ ] Decidir si el devcontainer se prueba ahora (Docker 29 listo) o se difiere.