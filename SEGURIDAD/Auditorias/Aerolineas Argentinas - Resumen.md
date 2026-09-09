---
title: "Aerolineas Argentinas - Resumen"
type: note
status: active
tags:
  - seguridad
  - auditoria
  - aerolineas
  - resumen
  - web
  - api
aliases:
  - "Auditoria Aerolineas"
  - "Aerolineas resumen"
created: 2026-09-09
updated: 2026-09-09
---

# Aerolineas Argentinas — Resumen

> [!INFO] Fuente
> `~/pages/aerolineas-argentinas/` — README, SUMMARY, `findings/01-07`, `research/reports/aerolineas-argentinas-research-2026-08-29.md`.
> Fecha audit: 29 Ago 2026, ultima verificacion 30 Ago 2026. 121 peticiones HAR analizadas.

---

## 1. Alcance en una linea

Web + API + CMS interno de Aerolineas Argentinas: `www`, `api`, `content.services`, CDN Azure y CloudFront.

| Dominio | Rol | Dato clave |
|---|---|---|
| `www.aerolineas.com.ar` (20.7.240.35) | Frontend SPA (Express SSR, Azure WAF) | Bypass WAF por `User-Agent` |
| `api.aerolineas.com.ar` (20.7.240.35) | API gateway | CORS `*`, JWT sin auth, metodos PUT/DELETE |
| `content.services.aerolineas.com.ar` (34.149.238.238) | CMS Drupal 8 / PHP 7.4.13 | EOL, sin auth, CORS `*` |
| `cms.services.aerolineas.com.ar` (10.200.91.4) | CMS interno | IP privada filtrada en JS |
| `www2`, `ftarplus` | Legacy ASP / intranet | Superficie vieja sin auditar |

---

## 2. Hallazgos (lo esencial)

| # | Hallazgo | Severidad | Estado | Idea en una frase |
|---|----------|-----------|--------|-------------------|
| 1 | PII sin auth `GET /v3/loyalty/members/{id}` | 🔴 | ✅ Parcheado (~24h, ahora 401) | IDOR: cualquier ID devolvia DNI, email, telefono, direccion |
| 2 | CORS `*` + `Credentials: true` | 🔴 | Activo | Cualquier web podia leer respuestas con sesion del usuario |
| 3 | Drupal 8 + PHP 7.4.13 + nginx 1.14.2 EOL | 🔴 | Activo | CMS interno sin auth, con cache poisoning posible |
| 4 | PUT/DELETE + CORP `cross-origin` | 🔴 | Activo | Preflight permite borrar/modificar localizaciones y canal `WEB_AR` |
| 5 | Fuga infra (Sabre PCC, gateway, UUID v1, headers Envoy/Azure) | 🟠 | Activo | Config de pagos + `x-api-server-segment: legacy` expuestos |
| 6 | Headers: sin CSP enforce, CSP reporta a Mercado Libre, sin Referrer-Policy | 🟡 | Parcial (HSTS agregado) | Telemetria y fugas a terceros |
| 7 | JWT privilegiado sin auth `GET /auth/token` | 🔴 Critica | Activo | JWT RS256 `catalog:admin loyalty:admin` sin credenciales |
| 8 | `client_secret` embebido en APK publico (Lab 06 Sep) | 🔴 Critica | Activo | El secreto del hallazgo 7 esta en el bundle JS de la app |

Cadena que hay que entender: **WAF filtra por UA → `/auth/token` da JWT admin → CORS `*` lo hace usable cross-origin → `/campaign/campaigns` devuelve 360 campanas con `keyPsw`**.

---

## 3. Como se verifico (para estudiar)

```bash
# PII (hoy debe dar 401)
curl -s "https://api.aerolineas.com.ar/v3/loyalty/members/65960130"
bash ~/pages/aerolineas-argentinas/tests/idortest.sh 65960130 65960150

# CORS
curl -sv "https://api.aerolineas.com.ar/v3/loyalty/members/65960130" 2>&1 | grep -i "access-control-allow"
bash ~/pages/aerolineas-argentinas/tests/cors-test.sh

# Headers / Drupal
curl -sv "https://content.services.aerolineas.com.ar/api/footer_content" 2>&1 | grep -iE "x-generator|x-powered-by|server|x-drupal"
bash ~/pages/aerolineas-argentinas/tests/headers-test.sh
bash ~/pages/aerolineas-argentinas/tests/deep-audit.sh
```

Recon previo: httpx + nmap + nuclei (9 headers faltantes, 9 misconfig, 0 CVE tras WAF) + ffuf (294 endpoints en `content.services`).

---

## 4. Que falta / proximos pasos

- [ ] Verificar parcheo CORS (API + content.services)
- [ ] Migrar Drupal 8 / PHP 7.4
- [ ] Cerrar PUT/DELETE o exigir auth, CORP a `same-origin`
- [ ] UUID v1 → v4, CSP report-only → enforce en dominio propio
- [ ] Rotar `client_secret` movil (comprometido) y pasar a PKCE sin secreto
- [ ] Retest post-remediacion

---

## 5. Para conectar con teoria

- [[SEGURIDAD/IDOR]] — caso 01-pii-exposure
- [[SEGURIDAD/CSRF]] — caso 02-cors + cookie `SameSite=None`
- [[SEGURIDAD/Vulnerabilidades Web]] — Drupal EOL, PUT/DELETE, UUID predecible
- [[SEGURIDAD/Reconocimiento]] — ffuf, nuclei, httpx del reporte
- [[SEGURIDAD/Reportes]] — formato findings → evidencia → remediacion

---

## Ver tambien

- [[SEGURIDAD/Auditorias/00 - INDICE Auditorias]]
- [[SEGURIDAD/Auditorias/Android Lab - Resumen|Android Lab]] (el secreto embebido)
- Originales: `~/pages/aerolineas-argentinas/SUMMARY.md`, `~/pages/aerolineas-argentinas/findings/`
