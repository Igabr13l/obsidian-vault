---
title: "Aerolineas Argentinas - Catalogo completo"
type: note
status: active
tags:
  - seguridad
  - auditoria
  - aerolineas
  - catalogo
  - vulnerabilidades
aliases:
  - "Catalogo Aerolineas"
  - "Todas las vulnerabilidades Aerolineas"
created: 2026-09-09
updated: 2026-09-09
---

# Aerolineas Argentinas — Catalogo completo (sin omisiones)

> [!INFO] Fuente
> `~/pages/aerolineas-argentinas/findings/` (01–07 + validation AA-01–AA-16) y `~/pages/research/reports/aerolineas-argentinas-research-2026-08-29.md` (R-01–R-17 + Auth0).
> El [[SEGURIDAD/Auditorias/Aerolineas Argentinas - Resumen|Resumen]] es la lectura corta; esto es el inventario total.
> Estados a 30 Ago 2026 salvo el hallazgo movil (06 Sep 2026).

---

## A. Findings formales (`findings/`)

| ID | Hallazgo | Sev | Estado | Una frase |
|----|----------|-----|--------|-----------|
| 01 | PII sin auth `GET /v3/loyalty/members/{id}` | 🔴 | ✅ Parcheado (401) | IDOR: DNI, email, telefono, direccion sin sesion; `membershipCode` en URL |
| 02 | CORS `*` + `Credentials: true` (API + content.services) | 🔴 | Activo | Cualquier origen lee respuestas con sesion del usuario |
| 03 | Drupal 8 + PHP 7.4.13 + nginx 1.14.2 EOL | 🔴 | Activo | CMS interno sin auth (Drupalgeddon, RCE PHP<8) |
| 04 | PUT/DELETE + CORP `cross-origin` + `Expose-Headers: Content-Disposition` | 🔴 | Activo (app 403/302, preflight permite) | Modificar canal `WEB_AR` (gateway, merchantId) o borrar localizaciones |
| 05 | Fuga infra (Sabre PCC/NFA, gateway WORLDLINE, UUID v1, headers Envoy/Azure, regex DNI/pasaporte) | 🟠 | Activo | Config de pagos + fingerprint `x-d2id` estable entre sesiones |
| 06 | Headers: sin CSP enforce (report-only a Mercado Libre), sin Referrer/Permissions/COOP/CORP, trace New Relic a terceros | 🟡 | Parcial (HSTS agregado) | Telemetria y fugas a terceros |
| 07 | `client_secret` OAuth en APK publico (Lab Android 06 Sep) | 🔴 | Activo | `grep` al bundle → JWT admin minteable por cualquiera |

## B. Research activo (R-01–R-17)

| ID | Hallazgo | Sev | Estado | Una frase |
|----|----------|-----|--------|-----------|
| R-01 | CORS `*` + credenciales en `content.services` | 🔴 | Confirmada (7646 B exfiltrados) | = 02, reproducido con Origin falso |
| R-02 | Drupal/PHP EOL | 🔴 | Confirmada | = 03 |
| R-03 | PUT/DELETE via CORS | 🔴 | Confirmada (preflight) | = 04 |
| R-04 | UUID v1 `f79b5b20-…` predecible (GDS profile) | 🔴 | Confirmada | Enumeracion de perfiles por timestamp |
| R-05 | Cache poisoning (`x-drupal-cache: HIT`, `age` incremental, key debil por idioma) | 🔴/🟠 | Confirmada | Inyeccion `Accept-Language` → 109 KB |
| R-06 | Headers exponen infra (`cms.services → 10.200.91.4`, `legacy`) | 🟡 | Confirmada | `x-d2id`/`x-azure-ref` ya no aparecen (mitigacion parcial) |
| R-07 | 294 endpoints por ffuf (`/api/auth/*`, `/api/v1/users/admin`, `/api/graphql`, `/api/swagger.json`) | 🟠 | Confirmada | 403 en API, CORS abierto en interno |
| R-08 | Segmento `legacy` sin auditar | 🟡 | Activo | Ver 05 |
| R-09 | PHP parameter pollution (`langcode[0]=`) | 🟡 | Activo | Array injection / fingerprint Laravel |
| R-10 | CORP `cross-origin` explicito | 🟡 | Activo | Ver 04 |
| R-11 | `operationType` CRUD arbitrario (`C/R/U/D/X`) en loyalty `evaluateDestinationMemberEligibility` | 🟡/🟠 | Confirmada | Sin restriccion (WAF solo frena inyeccion clasica) |
| R-12 | HTTP/3 con QUIC antiguo (`h3-29`) | 🟢 | Activo | Superficie menor |
| R-07n | WAF bypass por `User-Agent` (`curl` 403, `Mozilla` 200 → `/health-check`, `/auth/token`, `sublos/login` 447 kB) | 🔴 | Confirmada | Toda la cadena nocturna cuelga de aqui |
| R-08n | JWT privilegiado sin auth (`GET /auth/token`, RS256, `catalog:admin loyalty:admin`, 24 h) | 🔴 | Confirmada | `fresh_token.json` |
| R-09n | CORS `*` en `api` + `www` (Express) | 🔴 | Confirmada | Hasta en respuestas 401/404 |
| R-10n | 359 campanas con `keyPsw` (26 chars base32) + patrones BIN/CUIL/cupon | 🟠 | Confirmada | `campaigns_evidence.json` |
| R-11n | Bundles localizacion 3.4 MB (288 bundles, URLs internas, reglas) | 🟡 | Confirmada | |
| R-12n | Hosts internos en `client.cbc5d9f6.js` (`api-gateway.backend.svc.cluster.local`, `ar-cms-cache.backend`) + key `ipapi.co` | 🟡 | Confirmada | Util para SSRF si existe |
| R-13n | Subdominios legacy/intranet (`www2` ASP plano, `ftarplus` intranet, Sabre externo) | 🟡 | Activo | |
| R-14n | Clickjacking en `www` (sin XFO/CSP con UA Mozilla) | 🟡 | Confirmada | Login y booking embebibles |
| R-15n | Escritura admin con JWT no autenticado (`POST/PUT /v1/campaign/campaigns` → 400 negocio, no 401) | 🔴 | Confirmada | `admin_write_evidence.txt` |
| R-16n | IDOR booking / `operationType` + rate-limit (IP block tras ~100 reqs) | 🟠 | Confirmada | `loyalty_evidence.txt` |
| R-17n | Usernames staff Drupal via CMS (`roxana.beresaga`, `santiago.quinteiro`, `ana.prestera`; 12 IDs incl. `user/1`) | 🟠 | Confirmada | Phishing + stuffing; CORS `*` |
| R-Auth | Auth0: `client_id` expuesto en redirect, tenant `aerolineas-test` con error, `/dbconnections/*` mapeados, `client_secret` no expuesto en web | 🟡/info | Activo | El secreto aparecio despues en el APK (07) |

## C. Validation 30 Ago (AA-01–AA-16)

| ID | Estado | Nota |
|----|--------|------|
| AA-01 PII | ✅ Parcheado | 401/403 |
| AA-02 CORS | 🔴 Sigue activo | API + frontend |
| AA-03 GDS sin auth | ✅ Bloqueado (WAF) | |
| AA-04–07 infra catalogo | ✅ Bloqueado (WAF) | Detras de headers de navegador |
| AA-08 CORS en JS estatico | ✅ Bloqueado (WAF 403) | |
| AA-09 HSTS | ✅ Parcheado | `max-age=31536000; includeSubDomains; preload` |
| AA-10 tech expuesta | 🟡 Parcial | API oculta tras Gateway; frontend sigue `nginx`+`Express`+`etag` |
| AA-11 PUT/DELETE | ✅ Bloqueado (WAF 403) | Preflight CORS sigue permisivo |
| AA-12 OPTIONS | ✅ Bloqueado (WAF 403) | |
| AA-16 headers | 🟡 Parcial | Sin CSP/Referrer/Permissions/COOP/CORP/XFO en frontend |

## D. Controles positivos / limites

- `GET /v3/loyalty/members/{id}` con machine-token → 401 correcto (requiere user JWT).
- Drupal paths (`jsonapi`, `admin`, `node/`, `user/`) → 403 Varnish.
- `api` con rate-limit/IP block temporal; `content.services` sin rate-limit.
- Nuclei CVE: 0 matches tras WAF. `sourcemaps` 404 (sin filtracion).

---

## Ver tambien

- [[SEGURIDAD/Auditorias/00 - INDICE Auditorias]] · [[SEGURIDAD/Auditorias/Aerolineas Argentinas - Resumen|Resumen]] · [[SEGURIDAD/Auditorias/Android Lab - Resumen|Android Lab (07)]]
- Fuentes: `~/pages/aerolineas-argentinas/findings/`, `~/pages/research/reports/aerolineas-argentinas-research-2026-08-29.md`, `~/pages/research/scripts/recon/`
