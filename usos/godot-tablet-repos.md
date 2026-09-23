# GODOT EN LA TABLET + REPOS GITHUB + PLANTILLAS

> Guía de operación (23-sep). Respuestas a: ¿funciona Godot en la tablet?
> ¿Cómo se conecta a GitHub? ¿Puedo usar un repo como guía/plantilla base?

## 1. ¿Godot corre en la tablet? — SÍ, editor oficial Android

- **Existe el editor oficial para Android desde 2023** (Godot 4.x). Es la MISMA
  versión que el PC (4.7.2 stable actual).
- **Instalación** (Lenovo Tab M11, arm64):
  - **Vía oficial**: descargar `Godot_v4.7.2-stable_android_editor.apk` (~656MB)
    de `github.com/godotengine/godot-builds/releases` (Assets) o de
    `godotengine.org → Download → Android`. Instalar con "fuentes desconocidas".
  - **Vía F-Droid**: `org.godotengine.editor.v4` (más pequeña ~145MB, versión
    4.5.1 más vieja, sin exportar).
  - ⚠️ APK del editor Android **NO exporta proyectos** (exportar = PC/USB).
- **Para sacar APK jugable**: se exporta desde el editor del PC. Después ese
  APK se instala en la tablet/teléfono. El editor Android crea JUGABLES, no
  EXPORTA release.
- **Límites del editor Android (oficiales)**:
  - Solo **GDScript** (sin C#/Mono).
  - Render **Forward+ = mal rendimiento** → usar Compatibilidad (2D va perfecto).
  - UI **no optimizada para táctil**: con ratón+teclado (tu setup) es el caso
    recomendado por la documentación.
  - Editor en "estado experimental" pero funcional para crear/editar 2D.

## 2. Cómo apunto a los repos de GitHub desde la tablet

Dos métodos — elegir según comodidad:

### Método A — Git + push (recomendado: flujo completo)
1. En Termux (terminal en la tablet): instalar git y gh:
   ```
   pkg install git gh
   gh auth login
   ```
2. En la tablet: `cd ~/storage` y `git clone <tu-repo>` (p. ej.
   `game-profiles`, o el repo de plantilla-base del §4).
3. Abrir el proyecto en el editor Godot Android → editar → en Termux:
   `git add . && git commit -m "..." && git push`.
4. En el PC/USB: `git pull` → Godot abre igual → exporta APK final.
   → Uso completo del "GitHub como docker personal" (estacion-editor.md).

### Método B — Syncthing (sync simple, mínimo git)
1. La carpeta del proyecto ya está en `Freelance Hub`/espejo (ver checklist piloto).
2. Editar en la tablet con Godot; Syncthing sincroniza al PC; el PC hace commit/
   push/export. Cero git en la tablet.

> Recomendación: uso **B para probar** (arranca sin config de git en la tablet) y
> A cuando el repo de plantillas esté montado (ya que A es el flujo "remoto real").

## 3. ¿El repo-gal como "guía para crear todos los que necesito"?

- **Sí**: se puede tener un repo **plantilla-base** con esqueletos de proyectos
  Godot siguiendo el patrón clave de GitHub:
  `github.com/<user>/godot-templates` (o una carpeta `templates/` en game-profiles).
- **Cómo funciona**: cada plantilla es un **proyecto Godot listo para clonar**,
  con su `project.godot`, escenas base, scripts de ejemplo y `README.md`
  explicando qué es y qué contiene.
- **Flujo de creación de un juego nuevo**:
  1. En GitHub web: clonar la plantilla → "Create repository from template"
     (GitHub lo permite: template repo).
  2. O en la tablet/PC: clonar la carpeta base → renombrar → abrir en Godot →
     editar como proyecto propio.
- **Templates propuestos (según lo que vas a crear)**:

| Plantilla | Para qué | Contenido base |
|---|---|---|
| `blank-2d` | Arrancar de cero sin nada | project.godot (Compatibility), Main escena + icono, README |
| `platformer-2d` | Juegos de plataformas (tipo dodge-the-creeps avanzado) | Character (velocity, salto, gravedad), TileMap de muestra, cámara seguidora |
| `top-down-2d` | Rogue-like / survivor (tienes Top_Down_Survivor assets) | Player + movimiento 8 dir, Enemy básico, area de daño |
| `gui-app` | Menús/config (pantallas) | Base de Menú + config + escena App |
| `3d-basic` | (opcional) escena 3D mínima | Camera3D + CharacterBody3D + suelo |

- **Decision** (a definir con Samuel): crear repo nuevo `godot-templates` (privado)
  o carpeta `templates/` dentro de `game-profiles`. Al clonar desde GitHub, el
  repo-template permite "new from template" en la web — recomendado.

## 4. Estado actual de los repos (verificado 23-sep)

- `game-profiles` (privado): **vacío en GitHub** (0 commits); local tiene
  `Laboratiorio Godot` con escenas (`Escenas/nivelUno.tscn`) + assets
  (`Rec. Principales/` con charzera, City Background, cat sprite, terrain…).
- `estacion-operaciones` (privado): el docker personal (devcontainer + Copilot).
- Otros: freelance-pipeline, automatizaciones, ghostink, vanilla-os-griezz, dotfiles.

## 5. Checklist para arrancar Godot en la tablet (pendiente S.M.)

- [ ] Descargar APK editor Android 4.7.2 en el PC (GO: descargar de godot-builds).
- [ ] Enviar APK a la tablet (KDE Connect / adb) e instalar (fuentes desconocidas).
- [ ] Decidir: repos método A (git+gh en Termux) o B (Syncthing). → A.
- [ ] Decidir: repo de templates nuevo vs carpeta en game-profiles.
- [ ] Montar 1-2 plantillas (blank-2d y platformer-2d) y versionarlas.
- [ ] Prueba: clonar plantilla en la tablet → abrir en Godot → editar → push → pull en PC.