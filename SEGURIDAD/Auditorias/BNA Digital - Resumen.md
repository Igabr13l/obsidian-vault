---
title: "BNA Digital - Resumen"
type: note
status: active
tags:
  - seguridad
  - auditoria
  - bna
  - resumen
  - web
  - api
aliases:
  - "Auditoria BNA"
  - "BNA Digital resumen"
created: 2026-09-09
updated: 2026-09-09
---

# BNA Digital — Resumen

> [!INFO] Fuente
> `~/pages/bna-digital/` — README, SUMMARY, `findings/` (7), `evidence/auditoria/` (6 informes).
> Fecha: 29 Ago 2026. Analisis no intrusivo (sin payloads ni credenciales).
> Inventario total sin omisiones: [[SEGURIDAD/Auditorias/BNA Digital - Catalogo completo|Catalogo completo]].

---

## 1. Alcance en una linea

Ecosistema BNA Digital + operador Red Link. Infra: Volterra ADC (Fastly) + Envoy, edge San Pablo. Cert Sectigo wildcard `*.bna.com.ar` (19-mar-2026 → 03-oct-2026).

| Dominio | Rol |
|---|---|
| `digital.bna.com.ar` | SPA React (`/desktop`), API `/api/v1/execute` |
| `bnanet.bna.com.ar` | Web legacy IBM/Lotus (`LtpaToken2`, Dreamweaver JS) |
| `oauth.bna.com.ar` / `api.bna.com.ar` | SSO / gateway REST |
| `hb.redlink.com.ar`, `bancosbna.api.redlink.com.ar`, `analytics.redlink.com.ar` | Home banking clasico, APIs corporativas, analytics (Red Link S.A.) |

---

## 2. Hallazgos (lo esencial)

| # | Hallazgo | Severidad | Idea en una frase |
|---|----------|-----------|-------------------|
| 1 | CORS refleja `Origin` arbitrario + `Allow-Credentials: true` en `/api/v1/execute` | 🔴 | `evil.com`, `null` y similares son reflejados; `configuration.listConfiguration` responde 200 sin auth con 720 claves |
| 2 | Criptografia en navegador (`crypto-js`, `modo.cvv.encryption.password`) | 🔴 diseno | Clave simetrica CVV distribuida en claro; firma `_std_` = HMAC con el propio bearer como clave, OTP fuera del HMAC |
| 3 | `bnanet` legacy IBM/Lotus, XHTML 1.0, `iso-8859-1` | 🔴 superficie | Stack de ~20 anos sin controles modernos |
| 4 | HSTS solo en OPTIONS, no en GET/POST | 🟠 | Navegacion normal sin HSTS → SSL stripping |
| 5 | Cookies `SameSite` inconsistentes (`None` + `Strict` + sin definir) | 🟠 | La cookie `SameSite=None` es la que materializa el ataque del hallazgo 1 |
| 6 | Sin `security.txt` | 🟠 | Sin canal de reporte |
| 7 | Fingerprinting (`fingerprint2.js`, `vubrowserfp.js`) + beacon a `analytics.redlink.com.ar` | 🟠 privacidad | Huella + referrer a tercero, posible roce con Ley 25.326 |

Reporte OWASP aparte (`evidence/auditoria/bna-digital-vulnerability-report-2026-08-29.md`): 27 vulns (8 criticas, 10 altas, 9 medias) en OWASP Top 10:2025. Casos historicos que contextualizan: Ortmann 2019 (manipulacion cotizacion dolar desde frontend) y BNA+ CVU 2022 (validacion incompleta de montos).

### Transaccional (del analisis HAR — lo mas grave)

Fuente: `evidence/auditoria/bna-digital-har-security-analysis-2026-08-29.md` (CRIT/HIGH con evidencia de trafico real autenticado via HAR aportado).

| ID | Hallazgo | Severidad | Idea en una frase |
|----|----------|-----------|-------------------|
| CRIT-02 | Soft-token reutilizado y propagado | 🔴 | El `_otp` (`SOFT_TOKEN@<6 digitos>`) viajo sin cambios en 29 requests durante ~102.000 s, a endpoints que no lo necesitan (widgets, inversiones, prestamos, tarjetas) → fuga a logs/APM y replay en ventana TOTP |
| HIGH-01 | `ignore.sign.activities` excluye operaciones sensibles de firma | 🟠 → 🔴 si se confirma | `modo.sendTransfer`, `modo.makePayment(QR)`, `modo.changePhone`, `core.cancelTransaction`, `corporate.bulkTransfer.fileUploader`, `echeq.emitMultiple.fileUploader`… si el backend no firma por otro lado, importe/cuenta/beneficiario son alterables |
| HIGH-08 | Firma `_std_` = HMAC con el propio bearer como clave, OTP excluido | 🟠 → 🔴 si backend confia en ella | Quien tenga el bearer recalcula la firma tras modificar el payload; el OTP no queda ligado a importe/cuenta/beneficiario |
| HIGH-06 | `ticket.get` / `approval.transactions.searchById` devuelven payload interno completo | 🟠 | `amount`, `account`, `documentNumber`, `_std_`, fingerprints, IP, comprobante Base64 32 KB — reexpone el esquema de integridad |
| HIGH-07 | BOLA/IDOR candidato por `transactionId` (hex 32, alta entropia) | 🟠 candidato | Si el backend no verifica titularidad, un ID filtrado (notificacion, log, soporte) lee transacciones ajenas |
| HIGH-03 | JWT HS256 sin `exp/aud/iss/jti` | 🟠 | Mitigado parcial por sesion backend de 5 min; riesgo si algun servicio valida solo la firma |
| HIGH-05 | Umbrales antifraude expuestos (`otpBna.maxAttempts`, `retail.factor.amount`, firmas por defecto) | 🟠 | Permite calibrar fraude por debajo de los controles |

Matiz importante: `session.get`, saldos y operaciones devolvieron 401 sin bearer — la exfiltracion autenticada **no** quedo demostrada; pendiente plan T-01..T-19 con dos cuentas de laboratorio.

---

## 3. Como se verifico

```bash
bash ~/pages/bna-digital/tests/cors-test.sh       # reflejo Origin + credenciales
bash ~/pages/bna-digital/tests/headers-test.sh    # HSTS/CSP en GET

# Manual
curl -sI -X OPTIONS https://digital.bna.com.ar/api/v1/execute -H "Origin: https://evil.com"
curl -s -X POST https://digital.bna.com.ar/api/v1/execute/configuration.listConfiguration -H "Origin: null"
```

Retests 29 Ago: CORS confirmado 23:10 UTC; HSTS/CSP ausentes en `GET /desktop` 21:59 UTC.

---

## 4. Que falta / proximos pasos

- [ ] CORS a allowlist + quitar credenciales donde no haga falta
- [ ] Config sensible con auth o allowlist minima; rotar `modo.cvv.encryption.password` y `plugin.check.apiKey`
- [ ] Mover crypto al backend; challenge transaccional server-side ligado a importe/cuenta/beneficiario
- [ ] HSTS en todas las respuestas, `SameSite` coherente, publicar `security.txt`
- [ ] Migrar `bnanet`; ejecutar plan T-01..T-19; retest post-remediacion

---

## 5. Para conectar con teoria

- [[SEGURIDAD/CSRF]] — hallazgo 1 + hallazgo 5 (la combinacion que lo hace explotable)
- [[SEGURIDAD/Vulnerabilidades Web]] — HSTS, security.txt, legacy EOL
- [[SEGURIDAD/HTTP Profundo]] — CORS preflight, cookies SameSite, HSTS
- [[SEGURIDAD/OSINT Profundo]] — `bo-prod` filtra topologia AWS, `digitaltest-token` expuesto en Firebase
- [[SEGURIDAD/Reportes]] — informes en `evidence/auditoria/`

---

## Ver tambien

- [[SEGURIDAD/Auditorias/00 - INDICE Auditorias]]
- [[SEGURIDAD/Auditorias/Android Lab - Resumen|Android Lab]] (BNA+ movil)
- Originales: `~/pages/bna-digital/SUMMARY.md`, `~/pages/bna-digital/findings/`, `~/pages/bna-digital/evidence/auditoria/`
