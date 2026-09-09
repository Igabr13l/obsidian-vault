---
title: "Android Lab - Resumen"
type: note
status: active
tags:
  - seguridad
  - auditoria
  - android
  - mobile
  - frida
  - resumen
aliases:
  - "Android RE resumen"
  - "Lab Android resumen"
created: 2026-09-09
updated: 2026-09-09
---

# Android Lab — Resumen (Aerolineas, BNA+, BAX)

> [!INFO] Fuente
> `~/pages/research/android-re/reports/INFORME-GENERAL.md` + reportes por app (06 Sep 2026).
> Lab: emulador `lab` API 34 arm64 userdebug + frida-server 17.17.0 + mitmproxy con CA de sistema. Fase dinamica **pre-login** (sin credenciales).

---

## 1. Tabla comparativa

| App | Paquete / ver | Stack | Crit | Alto | Medio | Bajo | Dinamico |
|---|---|---|---|---|---|---|---|
| Aerolineas | `net.aper.ARMobile` 2.11.2 | React Native + Hermes | 1 | — | 1 | — | ✅ completo pre-login |
| BNA+ | `com.banconacion.bnamas` 7.16.2 | Nativa Kotlin + Veritran packer | — | 1 | 1 | — | ⚠️ gate integridad |
| BAX (GCBA) | `com.gcba.bax` 1.3.0 | Flutter Dart AOT | 1 | 1 | 2 | 1 | ✅ completo pre-login |

---

## 2. Aerolineas — lo esencial

- APK 2.11.2 (code 2092102, SHA `aad30c1b…f60801cb`), React Native + Hermes, bundle JS 5.8 MB legible, minSdk 28 / targetSdk 36.
- 🔴 **`client_secret` OAuth en claro en `assets/index.android.bundle`**: con `grep` al APK se mina JWT RS256 (`catalog:admin loyalty:admin forms:admin`, ~24h). Replay con `curl -A okhttp/4.12.0` → 200 (con UA `curl` → 403: el WAF solo filtra UA). Con ese JWT, `/campaign/campaigns` → 200 con 360 campanas y `keyPsw`.
- Secuencia de arranque (sin login): `POST /v1/auth/token` → `GET content.services…/api/prohibit…` → catalogos `/v1/catalog/*` → `/v2/device/inbox?deviceCode=null` → Meta/Firebase. Headers app: `x-channel-id: MOBILE_AR`, `x-app-version: 2.11.0` (declara 2.11.0 siendo 2.11.2), UA `okhttp/4.12.0`.
- JWT: 11 permisos, 4 admin; `iss` = tenant Auth0 **de test** emitiendo para API prod. Bundle sweep limpio (sin Google keys ni hosts dev/staging): unico secreto el `client_secret`.
- 🟡 Token no persiste pre-login, pero hay key `@arsa/token-G` en SQLite **sin cifrar** → post-login caeria en claro.
- ThreatMetrix dormido pre-login. Manifest sano (`allowBackup=false`). Gate de update forzado 2.11.3 sin bypass.
- Backends: `api`, `content.services`, `checkin.../dx/ARCI`, `upgrade.plusgrade.com`, `ecommerceapi.assistcard.com`. CORS `*` confirmado en vivo.

## 3. BNA+ — lo esencial

- APK 7.16.2.52363 (SHA `5890f2fc…f0771411`), nativa Kotlin (~33.5k clases), strings cifrados, `io.jsonwebtoken` embarcado. SDKs: BlinkID (DNI), MODO pagos, Duktape (JS embebido), Sentry. Permisos: `READ_CONTACTS`, `CAMERA`, `FINE_LOCATION`, `NFC`, `READ_PHONE_STATE`.
- Defensa fuerte: Veritran + DexGuard + dex cifrado en `assets/` + RootBeer en `libtoolChecker.so`.
- 🟠 Gate de integridad server-side (SafetyNet): `ua599` (root/emulador) bypasseado con `root-emulator-hide.js`; `ua233` ("verifique internet") **no** bypasseado, cadena `B6.k → B6.l → kb.C → B6.U.o0/q0(233)` trazada.
- 🟡 `bnamas.redlink.com.ar` (45.233.70.116, cert RED LINK S.A.) con **pinning nativo** (inmune a hooks Java/OkHttp/Conscrypt).
- Storage: `vtuapp.db` 10 tablas solo-hashes; `secure_prefs.xml` cifrado correcto; fingerprint `DK` en claro (el spoof `VTgoldfish_arm64…Pixel 8` quedo persistido: el Build-spoof envenena su fingerprint pero no abre el gate). Frontera: Ghidra o equipo fisico + Magisk/PlayIntegrityFix.

## 4. BAX — lo esencial

- APK 1.3.0 (code 461, split 139 + 87 MB, SHA base `5beace07…cd775de`), Flutter Dart AOT (`libapp.so` 11 MB), vendor PairIP.
- Backends prod (`buenosaires.gob.ar`): `miba-api2`, `miba-sessions`, `login`, `front-verificador-generico…/bax`. Chat IA = BotMaker (`api.botmaker.com` + WebViews locales); biometria Sobio (`libsobio_dni 27 MB` + `libsobio_face 16 MB`); WebRTC, PDFium, ML Kit barcode, OWM (`weather?lat=&appid=`).
- 🔴 **4 proyectos Firebase embebidos** (prod + dev/qa/hml) con buckets y 4 claves `AIza…` + clave OpenWeatherMap concatenada en runtime → superficie dev/qa/hml alcanzable desde prod.
- Detalle 09 Sep (extraccion directa de `libapp.so`, valores completos solo en el lab local, aqui truncados): prod `…naH8` → `bax-produccion` (+ `bax-produccion.firebasestorage.app`); dev/qa/hml `…-wenI`, `…epRg`, `…KLXI` → `bax-mobile-{dev,qa,hml}.firebasestorage.app` (mapeo probable por pool de strings). OWM: URL `api.openweathermap.org/data/2.5/weather?lat=` + `appid=` concatenado en runtime (2 candidatos 32-hex sin contexto atribuible). Riesgo: una `AIza` sola no da admin, pero combinada con reglas abiertas en dev/qa/hml (tipico) permite leer/escribir Storage/Firestore de ambientes internos; OWM = consumo de cuota ajena. Verificar reglas por ambiente; nunca commitear los valores completos (esta nota y el repo son git).
- 🟠 `allowBackup` ausente (= true). GA en pantallas de login + leak i18n (`invalidPasswordMessage` en app 100% espanol).
- 🟡 Actividades Sobio "Demo" exportadas; 🟢 hosts miBA no-prod hardcodeados (no resuelven, solo naming).
- Login Keycloak miBA triple federacion, PKCE `plain`. Gate PairIP bypasseado (solo chequea installer). Frontera: post-login con cuenta miBA de test.

---

## 5. Remedios transversales (memorizar)

1. No embarcar secretos en apps publicas → PKCE / public client + Play Integrity; rotar lo comprometido.
2. No publicar referencias multi-ambiente en prod.
3. Restringir API keys por app + SHA.
4. Sin analytics de terceros en login. 5. `allowBackup=false` explicito.

---

## 6. Para conectar con teoria

- [[SEGURIDAD/Mobile Pentesting]] — apktool/jadx, Frida, pinning, backup
- [[SEGURIDAD/Reconocimiento]] — `find-endpoints.sh`, `apk-info.sh`
- [[SEGURIDAD/CSRF]] — CORS `*` visto tambien en trafico movil
- Scripts propios reutilizables en `~/pages/research/android-re/scripts/`: `okhttp-log-unpin.js`, `root-emulator-hide.js`, `bax-license-bypass.js`, `no-force-update.js`

---

## Ver tambien

- [[SEGURIDAD/Auditorias/00 - INDICE Auditorias]]
- [[SEGURIDAD/Auditorias/Aerolineas Argentinas - Resumen|Aerolineas]] · [[SEGURIDAD/Auditorias/BNA Digital - Resumen|BNA]]
- Originales: `~/pages/research/android-re/reports/` (INFORME-GENERAL + carpetas por app)
