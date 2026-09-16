# Infra: API con cuentas + App móvil — arquitectura

> La estación expone sus servicios por API con autenticación por cuenta. Ese
> mismo contrato (API + JWT) es el que consume la app móvil. Construir y
> consumir esta API = el aprendizaje que se vende a Conecty.

## 1. Patrón general

```
cliente (web/móvil/script)
   │  POST /api/auth/login  {"email","password","tenantId"} → {token JWT}
   │  GET /api/...  (cabecera Authorization: Bearer <token>)
   ▼
API gauzy (NestJS) → módulos (contactos, props, correo) → DB
```

- **JWT**: token firmado; no guarda sesión; cada petición lleva el token.
- **Tenant**: gauzy aisla datos por organización (multi-tenant). El freelancer
  es un tenant; sus datos no se mezclan.
- **Roles**: SUPER_ADMIN (Samuel) vs EMPLOYEE (vista limitada, futuro).

## 2. Qué se expone (según los flujos documentados)

| Recurso | Uso |
|---|---|
| `/api/contacts` | fichas de contacto (conecty, imerit, …) |
| `/api/organization-contact` | relaciones empresa/contacto |
| `/api/proposals` + `/api/pipeline` | estado de candidaturas (cola/enviada/respondida) |
| `/api/email-history` | correo enviado/recibido por contacto |
| `/api/goals` / ideas | banda de ideas (residente) |
| `/api/tasks` | pendientes del freelancer (universidad, seguimientos) |

> Cualquier recurso nuevo propio (scripts, macros) se añade como módulo gauzy
> (patrón plugin).

## 3. Cliente móvil (fase F6)

- Primera versión: **webapp móvil** (misma Angular en responsive) — mínimo
  esfuerzo, mismo API.
- Luego app nativa (React Native/Flutter) si el interés lo justifica.
- Login por cuenta: email + contraseña → token → vista "mis candidaturas,
  mis contactos, pendientes".
- Pruebas SOLO con datos ficticios/sandbox.

## 4. Acceso remoto

- Local: `http://localhost:4200` (web) / `localhost:3000/api` (API).
- Remoto propio: vía **Tailscale** (ya instalado: PC fedora 100.99.115.32 +
  REDMI 100.114.237.2). Nada expuesto a internet público por ahora.

## 5. Aprendizaje ruta (para Conecty)

1. Arrancar API+web (Docker) y entrar por UI (cuenta admin).
2. Curl a la API: `login` → token → `GET /api/contacts` (ver el flujo JWT).
3. Crear un recurso/contacto desde la API (INSERT via REST).
4. Repetir el mismo acceso desde un script Python/curl (cliente sin UI).
5. Conectarlo desde el móvil por Tailscale → misma API, otra pantalla.
6. Material para Conecty: demo de "sistema de automatización con API + cuentas
   + contenedores + remoto".

## 6. Seguridad y reglas

- Credenciales nunca al repo/no a la nube (tenant local).
- JWT: corto plazo + refresh (seguir patrón gauzy); no loggear tokens.
- Los scripts actuales (griezz-*) quedan fuera del repo público; el repo lleva
  solo código de la estación + docs + `.env.example`.