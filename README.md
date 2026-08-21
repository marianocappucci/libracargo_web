# LibraCargo — Sitio Web / Landing

Landing de marketing de [LibraCargo](https://libracargo.com.ar), el vertical de
**agencias de cargas** de la familia Libra. Mismo patrón que las otras seis
landings: HTML estático servido por nginx en un contenedor Docker, con `/docs/`
gateado por un login real contra la instancia del cliente.

## Qué se edita dónde

🔴 **Dos de los tres pedazos de este sitio NO se editan acá.**

| Qué | Dónde vive | Cómo se actualiza |
|---|---|---|
| `public/index.html` | **Este repo** | A mano |
| `public/css/style.css` | `libra-web-kit` → `templates/style.css.template` + `site_css_tokens.SITES["libracargo"]` | `scripts/generate_css.py` |
| `public/docs/*.html` | `libra-web-kit` → `docs_content/libracargo/*.html` + `docs_pages` + `docs_sidebars` | `scripts/generate_docs.py` |

Los generados llevan un encabezado que lo dice. Editarlos acá funciona hasta la
próxima corrida del generador, que los pisa sin avisar.

```bash
cd ~/proyectos/libra-web-kit
.venv/bin/python scripts/generate_css.py
.venv/bin/python scripts/generate_docs.py
```

## Estructura

```
public/
  index.html          — Landing completa
  css/style.css       — GENERADO por libra-web-kit
  docs/               — 14 páginas, GENERADAS por libra-web-kit (gateadas por login)
auth/                 — Backend FastAPI del login de /docs/
  app.py              — 20 líneas de config sobre libra_web_kit.docs_auth
  Dockerfile          — python:3.12-slim
  requirements.txt
Dockerfile            — FROM ghcr.io/marianocappucci/libra-nginx-web (imagen compartida)
docker-compose.yml    — web (8100:80) + auth (interno), red stack_stack-net
.github/workflows/
  deploy.yml          — llama al reusable workflow de libra-web-kit
  alerta-ci.yml       — avisa por Telegram si el deploy sale en rojo
```

## Desarrollo local

```bash
docker compose build
docker compose up -d
```

Sirve en `http://localhost:8100`. Requiere la red externa `stack_stack-net`; en
el VPS ya existe, en local se crea con `docker network create stack_stack-net`.

## El login de /docs/

El backend `auth/` no tiene tabla de usuarios propia: valida en tiempo real
contra `https://{subdominio}.libracargo.com.ar/auth/verify`, el endpoint
server-to-server de la app, con el secreto compartido `DOCS_AUTH_SECRET`. Como
LibraCargo no tiene un backoffice con `/api/clientes-publicos`, el login pide el
**subdominio** por texto en vez de un `<select>`.

Rate limiting incluido en el kit: 5 intentos fallidos por IP cada 15 minutos.

## Variables de entorno (VPS)

| Variable | Para qué |
|---|---|
| `DOCS_AUTH_SECRET` | El mismo valor que en el `.env` de cada instancia de LibraCargo. Si difieren, el login de `/docs/` no valida nunca |
| `DOCS_SESSION_SECRET` | Firma la cookie de sesión de `/docs/` |
| `APEX_DOMAIN` | `libracargo.com.ar` |

## Relacionado

- Producto: [libracargo](https://github.com/marianocappucci/libracargo)
- Kit compartido: [libra-web-kit](https://github.com/marianocappucci/libra-web-kit)
- Mismo patrón: `contalibra_web`, `restolibra_web`, `gestiolibra_web`,
  `medlibra_web`, `ventalibra_web`, `libradesk_web`, `libraclub_web`
