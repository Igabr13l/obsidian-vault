---
title: "00 - Indice Auditorias"
type: note
status: active
tags:
  - seguridad
  - auditoria
  - indice
  - navegacion
aliases:
  - "Indice Auditorias"
  - "Resumenes auditorias pages"
created: 2026-09-09
updated: 2026-09-09
---

# 00 - Indice Auditorias

> [!INFO] Fuente
> Resumenes de lectura del proyecto `/Users/igabr13l/pages/` (29 Ago – 06 Sep 2026).
> Todo el analisis fue no intrusivo / pre-login, sin credenciales reales ni exfiltracion de datos personales.
> Los documentos originales viven en `pages/`; estas notas son solo la capa de lectura para Obsidian.

---

## Orden de lectura sugerido

| # | Nota | Que aprendes | Tiempo |
|---|------|--------------|--------|
| 1 | [[SEGURIDAD/Auditorias/Proyecto Pages - Mapa general\|Proyecto Pages - Mapa general]] | Mapa del proyecto: que contiene cada carpeta, donde esta la evidencia, que tests existen | 5 min |
| 2 | [[SEGURIDAD/Auditorias/Aerolineas Argentinas - Resumen\|Aerolineas Argentinas - Resumen]] | Auditoria web/API Aerolineas: PII, CORS, Drupal EOL, JWT sin auth | 10 min |
| 2b | [[SEGURIDAD/Auditorias/Aerolineas Argentinas - Catalogo completo\|Aerolineas - Catalogo completo]] | **Inventario total**: findings 01-07, research R-01-R-17, validation AA-01-AA-16 | referencia |
| 3 | [[SEGURIDAD/Auditorias/BNA Digital - Resumen\|BNA Digital - Resumen]] | Auditoria BNA + Red Link: CORS reflejado, crypto en navegador, legacy IBM/Lotus | 10 min |
| 3b | [[SEGURIDAD/Auditorias/BNA Digital - Catalogo completo\|BNA - Catalogo completo]] | **Inventario total**: 7 findings, BNA-01-11, HAR CRIT/HIGH/MED, OWASP 27, T-01-T-19 | referencia |
| 4 | [[SEGURIDAD/Auditorias/Android Lab - Resumen\|Android Lab - Resumen]] | Ingenieria inversa Android: Aerolineas (secreto embebido), BNA+ (Veritran), BAX (Firebase multi-ambiente) | 10 min |

---

## Estado de un vistazo

| Auditoria | Criticos | Altos | Medios | Bajos | Estado |
|---|---|---|---|---|---|
| Aerolineas web/API | 8 (1 parcheado) | 4 | 11 (1 parcheado) | 2 | 7 criticos activos |
| BNA Digital web | 3 (+8 en reporte OWASP) | 4 (+10 OWASP) | 4 (+9 OWASP) | 1 | Todos activos |
| Android Aerolineas 2.11.2 | 1 | — | 1 | — | Dinamico completo pre-login |
| Android BNA+ 7.16.2 | — | 1 | 1 | — | Limitado por gate integridad |
| Android BAX 1.3.0 | 1 | 1 | 2 | 1 | Dinamico completo pre-login |

---

## Conceptos que conectan con el vault

- [[SEGURIDAD/IDOR]] — hallazgo PII Aerolineas (`/v3/loyalty/members/{id}`)
- [[SEGURIDAD/CSRF]] — CORS `*` + credentials en Aerolineas y BNA
- [[SEGURIDAD/Vulnerabilidades Web]] — Drupal EOL, metodos PUT/DELETE, headers
- [[SEGURIDAD/Reconocimiento]] — subdominios, ffuf 294 endpoints, nuclei
- [[SEGURIDAD/Mobile Pentesting]] — Frida, pinning, backup, secretos en APK
- [[SEGURIDAD/Reportes]] — como estan estructurados los findings originales

---

## Ver tambien

- [[SEGURIDAD/00 - INDICE]]
- Fuentes: `~/pages/aerolineas-argentinas/`, `~/pages/bna-digital/`, `~/pages/research/android-re/reports/`
