---
title: "Proyecto Pages - Mapa general"
type: note
status: active
tags:
  - seguridad
  - auditoria
  - indice
  - mapa
aliases:
  - "Mapa pages"
  - "Estructura proyecto pages"
created: 2026-09-09
updated: 2026-09-09
---

# Proyecto Pages — Mapa general

> [!INFO] Fuente
> `~/pages/AGENT.md` + READMEs de cada repo. Lectura de 5 minutos para no perderse.

---

## 1. Que es cada carpeta

| Carpeta | Contenido | Entrada |
|---|---|---|
| `aerolineas-argentinas/` | Auditoria web/API (findings 01-07, tests, SUMMARY) | `README.md`, `SUMMARY.md` |
| `bna-digital/` | Auditoria BNA/Red Link (7 findings, 6 informes, 2 tests) | `README.md`, `SUMMARY.md` |
| `research/reports/` | Reporte investigacion activa Aerolineas 29 Ago (537 lineas, 17 vulns + 294 endpoints) | `aerolineas-argentinas-research-2026-08-29.md` |
| `research/android-re/` | Lab Android: apks, output, scripts Frida, reports por app | `README.md`, `reports/INFORME-GENERAL.md` |
| `research/skills/` | Metodologias propias: `vuln-research`, `web-app-testing`, `network-recon`, `reporting` | `research/skills/*/SKILL.md` |
| `public/` | Espejo publicable (sin evidencia sensible) | `public/README.md` |
| `AGENT.md` | Entorno, herramientas instaladas, bitacora de sesiones | raiz `~/pages/` |

---

## 2. Donde esta cada cosa (convencion)

- `findings/` → un .md por vulnerabilidad (descripcion + PoC + remediacion).
- `evidence/` o `evidence/har/` → evidencia cruda (HAR, JSON de tokens, capturas `.mitm`).
- `tests/` → scripts re-ejecutables (`cors-test.sh`, `headers-test.sh`, `idortest.sh`, `deep-audit.sh`).
- `research/android-re/reports/<app>/` → `triaje-inicial.md` + `analisis-dinamico.md` + evidencia.
- `research/android-re/scripts/frida/` → hooks (`okhttp-log-unpin.js`, `root-emulator-hide.js`, `bax-license-bypass.js`...).

---

## 3. Herramientas del entorno (resumen AGENT.md)

Recon: nmap, masscan, subfinder, amass, httpx, katana, ffuf. Vuln: nuclei, nikto, sqlmap, testssl, sslyze. Movil: adb, apktool, jadx, frida, objection, mitmproxy. Utilidad: jq, sqlite3. Sin: Burp/ZAP, hashcat, Docker.

---

## 4. Ruta de estudio con estos resumenes

1. Este mapa → 2. [[SEGURIDAD/Auditorias/Aerolineas Argentinas - Resumen|Aerolineas]] → 3. [[SEGURIDAD/Auditorias/BNA Digital - Resumen|BNA]] → 4. [[SEGURIDAD/Auditorias/Android Lab - Resumen|Android Lab]].
2. Si un concepto no se entiende (CORS, IDOR, HSTS...), saltar a la nota de teoria y volver.
3. Para profundizar, abrir el finding original citado al pie de cada resumen.

---

## Ver tambien

- [[SEGURIDAD/Auditorias/00 - INDICE Auditorias]]
- [[SEGURIDAD/00 - INDICE]]
