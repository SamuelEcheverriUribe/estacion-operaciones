# Infra: Contenedores (Docker) — plan de despliegue gauzy

> Estado: Docker aún NO instalado (autorizado por Samuel con sudo, 16-sep).
> Este doc define cómo será el flujo; se ejecuta en F2.

## 1. Por qué contenedores aquí

- La estación (gauzy: API NestJS + web Angular + Postgres) se monta de forma
  reproducible: misma infra en casa que en un servidor futuro.
- Aislamiento: nada de "instala en el sistema", todo dentro de imágenes.
- Aprendizaje: compose, volúmenes, redes, healthchecks → portátil a cualquier
  cliente (Conecty).

## 2. Stack objetivo (dev, liviano)

| Servicio | Imagen | Nota |
|---|---|---|
| API gauzy | ghcr.io/ever-co/gauzy-api | NestJS |
| Web UI | ghcr.io/ever-co/gauzy-webapp | Angular (nginx) |
| DB dev | SQLite embebida | sin servicio externo (modo dev) |
| DB prod | postgres:16 + redis | cuando se requiera escala |

> En dev se arranca API+Web con SQLite para aprender sin infra pesada.
> Prod/demo de gauzy trae `docker-compose.demo.yml` (imágenes prebuilt).

## 3. Mapa de conceptos (aprendizaje)

- **Imagen** = receta (código + dependencias + runtime).
- **Contenedor** = imagen corriendo, aislado.
- **Compose** = orquestar varios contenedores en un `docker-compose.yml`
  (servicios, redes, volúmenes, env).
- **Volumen** = datos persistentes fuera del contenedor (la DB vive aquí,
  sobrevive al `docker compose down`).
- **Red** = comunicación interna entre servicios (api→db) sin exponer todo al
  host.
- **Puerto** = entrada desde el host: `4200` (web), `3000/api` (API)…
- **Env** = configuración fuera del código (`.env`, no al repo).
- **Healthcheck** = prueba de que el servicio está vivo.

## 4. Comandos núcleo (para la operación diaria)

```bash
docker compose pull                       # trae imágenes
docker compose up -d                       # levanta en segundo plano
docker compose ps                          # estado
docker compose logs -f <servicio>          # logs en vivo
docker compose down                        # apaga (sin borrar volúmenes)
docker compose down -v                     # apaga Y borra volúmenes (¡datos!)
docker exec -it <contenedor> sh            # terminal dentro de un contenedor
```

## 5. Seguridad

- `.env` con credenciales NUNCA se commitea (`.gitignore`). Solo `.env.example`.
- Puertos solo en local (127.0.0.1) salvo que se decida exponer vía Tailscale.
- Datos de clientes (Suiza/HITL): no a la nube pública; esta estación es local
  (Tailscale para acceso remoto propio).
- Actualizar imágenes con `docker compose pull` (no `:latest` productivo, fijar
  tags).

## 6. Plan de ejecución (F2)

1. Instalar Docker Engine (sudo dnf) + plugin compose.
2. Probar `docker run hello-world` y un contenedor de ejemplo (nginx).
3. `docker compose -f docker-compose.demo.yml up` → gauzy en 4200
   (admin@ever.co / admin).
4. Ejercicio de aprendizaje: mapear puertos, subir bajada, volúmenes.
5. Documentar aquí el resultado real (restricciones, puertos, comandos útiles).