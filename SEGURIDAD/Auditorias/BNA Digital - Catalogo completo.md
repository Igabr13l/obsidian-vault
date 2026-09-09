---
title: "BNA Digital - Catalogo completo"
type: note
status: active
tags:
  - seguridad
  - auditoria
  - bna
  - catalogo
  - vulnerabilidades
aliases:
  - "Catalogo BNA"
  - "Todas las vulnerabilidades BNA"
created: 2026-09-09
updated: 2026-09-09
---

# BNA Digital — Catalogo completo (sin omisiones)

> [!INFO] Fuente
> `findings/` (7) + `findings/validation-report.md` (BNA-01–BNA-11) + `evidence/auditoria/`: HAR (CRIT-01/02, HIGH-01–08, MED-01–11), activo (1–12 + Sec.B), OWASP (C01–C08, A01–A10, M01–M09 = 27), plan T-01–T-19, recon pasivo, pruebas publicas.
> El [[SEGURIDAD/Auditorias/BNA Digital - Resumen|Resumen]] es la lectura corta; esto es el inventario total.
> Estados a 30 Ago 2026. Analisis no intrusivo / pre-login salvo HAR aportados.

---

## A. Findings formales (`findings/`, 7)

| ID | Hallazgo | Sev | Estado | Una frase |
|----|----------|-----|--------|-----------|
| 1 | CORS reflejado + config sensible anonima `/api/v1/execute` | 🔴 | Activo | `evil.com`/`null` reflejados; `configuration.listConfiguration` 200 anonimo con 720 claves |
| 2 | Crypto en navegador (`crypto-js`, `modo.cvv.encryption.password` al SDK MODO, `_std_` = HMAC con el bearer, OTP fuera del HMAC) | 🔴 diseno | Activo | Secreto simetrico distribuido en claro; firma recalculable con el bearer |
| 3 | `bnanet` legacy (IBM/Lotus, `LtpaToken2`, Dreamweaver JS, XHTML 1.0) | 🔴 superficie | Activo | ~20 anos sin controles modernos |
| 4 | HSTS solo en OPTIONS, no en GET/POST | 🟠 | Activo | SSL stripping en navegacion normal |
| 5 | Cookies `SameSite` inconsistentes (`None` + `Strict` + ausente) | 🟠 | Activo | La `None` materializa el ataque del hallazgo 1 |
| 6 | Sin `security.txt` (`digital` 200 catch-all, `bna.com.ar` 404) | 🟠 | Activo | Sin canal de reporte |
| 7 | Fingerprinting (`fingerprint2.js`, `vubrowserfp.js`, `nameSec.js` → `analytics.redlink.com.ar`) | 🟠 priv | Activo | Posible roce Ley 25.326 |

## B. Validation 30 Ago (BNA-01–BNA-11)

| ID | Estado | Nota |
|----|--------|------|
| BNA-01 CORS | 🔴 Confirmado activo | OPTIONS + GET reflejan |
| BNA-02 crypto-js | 🟡 Parcial | POST vacio ahora `COR020W`; scripts y config siguen |
| BNA-03 legacy | 🔴 Activo | |
| BNA-04 HSTS | 🔴 Activo | Test mejor configurado que prod |
| BNA-05 SameSite | 🔴 Activo | |
| BNA-06 security.txt | 🔴 Activo | |
| BNA-07 fingerprinting | 🔴 Activo | |
| BNA-08 headers faltantes | 🔴 Activo | `digital` parcial (HSTS/XFO/XCTO); `api/www/hb/bancosbna/analytics` casi sin nada |
| BNA-09 CORS en GET | 🔴 Activo | Tambien en POST `session.get` (401 pero refleja) |
| BNA-10 test publico `digitaltest-token` (Firebase/Fastly) | 🔴 Activo | Aislamiento por confirmar |
| BNA-11 fuga AWS (`bo-prod` → VPCE `us-east-1`) | 🔴 Activo | Topologia en DNS publico |

## C. Analisis HAR (CRIT-01/02, HIGH-01–08, MED-01–11)

| ID | Sev | Estado | Una frase |
|----|-----|--------|-----------|
| CRIT-01 | 🔴 | Confirmado | `modo.cvv.encryption.password` (27 chars) a todo cliente anonimo; revisar PCI DSS |
| CRIT-02 | 🔴 | Confirmado | `_otp` reutilizado en 29 requests a endpoints no relacionados (widgets, inversiones, prestamos); replay en ventana TOTP |
| HIGH-01 | 🟠→🔴 | Condicionado backend | `ignore.sign.activities` (45): `modo.sendTransfer/makePayment(QR)/changePhone`, `core.cancelTransaction`, `bulkTransfer.fileUploader`, `echeq.emitMultiple.fileUploader`, nominas, proveedores |
| HIGH-02 | 🟠 | Confirmado | HTML autenticado sin CSP ni anti-clickjacking |
| HIGH-03 | 🟠 | Confirmado | JWT HS256 sin `exp/iat/aud/iss/jti`; mitigado parcial por sesion 5 min |
| HIGH-04 | 🟠 | Confirmado | `modo.api.v3.debug=true`, `BeaconJs.enable.development.mode=true` en prod |
| HIGH-05 | 🟠 | Confirmado | Umbrales antifraude expuestos (`otpBna.maxAttempts`, `retail.factor.amount`, firmas por defecto, BioCatch) |
| HIGH-06 | 🟠 | Confirmado | `ticket.get`/`approval…searchById` devuelven `amount/account/documentNumber/_std_/fingerprints/IP` + comprobante 32 KB |
| HIGH-07 | 🟠 | Candidato | BOLA/IDOR por `transactionId` hex-32 (no enumerable, pero sin prueba de titularidad) |
| HIGH-08 | 🟠→🔴 | Confirmado diseno | `_std_` con bearer como clave, OTP excluido; ver Sec. transaccional del Resumen |
| MED-01 | 🟡 | Confirmado | JSON financiero sin `no-store` (`balance`, `products.list`) |
| MED-02 | 🟡 | Confirmado | Tabla 5 cookies `None/Strict`/ausente + `OKFBrFvy` sin verificar |
| MED-03 | 🟡 | Confirmado | `session.js` usa tiempo de aviso como expiracion; modal inmediato |
| MED-04 | 🟡 | Confirmado | `nameSec.js` manda URL+referrer sin codificar (detector phishing) |
| MED-05 | 🟡 | Confirmado | Backoffice `*.cc.bna.net`, Prisma/BioCatch, `Server: volt-adc`, `X-Volterra-Location: sp4-sao` |
| MED-06 | 🟡 | Confirmado | reCAPTCHA sin SRI ni CSP |
| MED-07 | 🟡 | Confirmado | Enlace `http://www.bcra.gov.ar` en config |
| MED-08 | 🟡 | Confirmado | `apple-touch-icon`/`favicon` → 200 shell SPA (rewrite alcanza estaticos) |
| MED-09 | 🟡 | Confirmado | `getRegisteredDevice` expone `deviceId/fingerprint/rooted/isJailbroken` (`pushToken: null` evita mas fuga) |
| MED-10 | 🟡 | Reclasificado (era Alto) | `plugin.check.apiKey` (130 chars) solo parametro del plugin de cheques; tratar como publica |
| MED-11 | 🟡 | Confirmado | PIN 4 digitos (`[0-9]{0,4}`) para alta/baja de usuarios corporativos; depende de lockout invisible |
| — | info | No-vuln | `messages.listMessages` 2.5 MB = placeholders, no PII; bearer nunca a Google/Gstatic |

## D. Analisis activo (1–12 + Sec. B)

Criticos 1–3 = findings 1–3. Altos 4–7 = findings 4–7. Medios propios:

| ID | Hallazgo | Una frase |
|----|----------|-----------|
| 8 | Errores verbosos (`API002E` + nombres de recurso en espanol) | Mapeo de superficie, sin stack trace |
| 9 | Stack heterogeneo (React + Lotus + IIS + nginx + Volterra + Envoy + GCP CDN) | Dificil aseguramiento uniforme |
| 10 | Datos a `analytics.redlink.com.ar` (45.233.68.25) | Fuera del perimetro del banco |
| 11 | `api.bna.com.ar` sin `nosniff` | Unico gateway sin el header |
| 12 🟢 | HEAD/GET `.map` inconsistente | Source maps NO expuestos (GET 404); HEAD enganosa |
| B | Retest 23:10 UTC + bundle `bb08b35d…` + pendientes | Base de T-01–T-19; 2FA/device-binding/firma corporativa existen pero neutralizados por CORS |

## E. OWASP Top 10:2025 (27: 8 crit + 10 altas + 9 medias)

| ID | Titulo |
|----|--------|
| C01 🔴 | IDOR en transferencias (`cbu_destino`, `monto`, `tipo_operacion` sin verificacion servidor) |
| C02 🔴 | Sin validacion servidor (paralelo Ortmann/Aerolineas) |
| C03 🔴 | Sin segmentacion por rol empresarial |
| C04 🔴 | Clave 6 digitos (10^6, sin lockout confirmado) |
| C05 🔴 | Clave local BNA+ 4 digitos |
| C06 🔴 | Transmision datos sensibles (rutas identificadas) |
| C07 🔴 | Posible SQLi (`cbu_origen/destino`, `monto`, `concepto`) |
| C08 🔴 | XXE en documentos (Webcomex) |
| A01 🟠 | Logica de negocio en frontend |
| A02 🟠 | Sesiones inseguras en migracion HB→BNA+ |
| A03 🟠 | CORS + headers |
| A04 🟠 | Info en errores |
| A05 🟠 | Stack sin parchar |
| A06 🟠 | Recovery de contrasena |
| A07 🟠 | MFA atado al mismo dispositivo (Token BNA+ in-app) |
| A08 🟠 | Sin integridad transaccional (sin HMAC server-side) |
| A09 🟠 | Integridad migracion de datos (saldos/historial) |
| A10 🟠 | SSRF a servicios externos |
| M01–M09 🟡 | Versiones expuestas · cookies · enumeracion usuarios · JS desactualizado · deteccion reactiva (Ortmann 165 tx sin deteccion) · monitoreo Red Link · integridad migracion · SIM swap · API corporativa SSRF |
| Esc 1–5 | IDOR transferencias · replica Ortmann · XSS+sesion · SSRF→core · XXE Webcomex |

## F. Recon pasivo + plan T

- Test Firebase publico, `bo-prod`→VPCE, `console.gcp` (Workforce/Entra), 20+ subdominios por catalogar, cert multi-dominio (`bna.com.es/.bo/.py/.uy`, etc.), TLS 1.2 OK, `gau` vacio. Positivo: Vault/Prometheus/ArgoCD/Grafana/Git **no** publicos.
- T-01–T-19: ejecutados solo checks publicos (T-01 falla, T-14 falla, matriz 401). Resto (T-02–T-13, T-15–T-19) pendientes con 2 cuentas lab. Cierre: escalar a CRIT si falla T-01/T-04/T-06/T-08/T-09/T-10/T-19.

---

## Ver tambien

- [[SEGURIDAD/Auditorias/00 - INDICE Auditorias]] · [[SEGURIDAD/Auditorias/BNA Digital - Resumen|Resumen]] · [[SEGURIDAD/Auditorias/Android Lab - Resumen|Android Lab (BNA+)]]
- Fuentes: `~/pages/bna-digital/findings/`, `~/pages/bna-digital/evidence/auditoria/`
