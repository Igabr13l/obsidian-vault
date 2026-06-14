---
title: "Reportes y Comunicacion"
type: note
status: active
tags:
  - seguridad
  - reportes
  - bugbounty
  - comunicacion
aliases:
  - Bug Reports
  - Vulnerability Disclosure
created: 2026-06-13
updated: 2026-06-13
---

# Reportes y Comunicacion

> [!INFO] Fuente
> Basado en las guias oficiales de HackerOne, Bugcrowd, y mejores prácticas de la industria.

---

## Estructura de un buen reporte

### 1. Resumen

```
Titulo: [Tipo] - [Ubicacion] - [Impacto breve]

Ej: "IDOR en /api/v1/invoices permite acceder a facturas de cualquier usuario"
```

### 2. Impacto

Que puede hacer un atacante con esto. Se concreto:

> ❌ "Se puede ver informacion de otros usuarios"
> ✅ "Cualquier usuario autenticado puede enumerar y descargar todas las facturas (nombre, direccion, tarjeta enmascarada, monto) de los ultimos 12 meses del resto de los usuarios simplemente incrementando el ID numerico en el endpoint GET /api/v1/invoices/{id}"

### 3. Pasos para reproducir (POC)

```
1. Crear cuenta A y B en target.com
2. Con cuenta A, generar una factura → recibis ID: INV-18472
3. Con cuenta B, reemplazar: GET /api/v1/invoices/INV-18472
4. Respuesta: datos completos de la factura de la cuenta A
```

### 4. Payload / PoC completo

Para bugs tecnicos, inclui el payload exacto:

```http
GET /api/v1/invoices/INV-18473 HTTP/1.1
Host: target.com
Cookie: session=...
```

### 5. Impacto potencial

- PII leak masivo si se automatiza
- Account takeover si se combina con otra vulnerabilidad
- Compliance (GDPR, PCI-DSS)

### 6. Remediation sugerida

```
- Implementar autorizacion por usuario en GET /api/v1/invoices/{id}
- Usar UUIDs no predecibles en vez de IDs secuenciales
- Rate limiting en endpoints sensibles
```

---

## El 7-Question Gate (de Claude-BugHunter)

Antes de enviar cualquier reporte, preguntate:

1. **¿Puedo reproducirlo consistentemente?** (no es intermitente)
2. **¿Es un安全问题 real?** (no es comportamiento esperado/documentado)
3. **¿Esta en scope?** (dentro del programa y los assets autorizados)
4. **¿No es un duplicado?** (revisaste triage/reportes previos?)
5. **¿Tiene impacto?** (no es cosmetico)
6. **¿Esta probado en el entorno correcto?** (produccion > staging)
7. **¿No contiene PII real?** (datos de prueba, sanitizados)

> [!IMPORTANT] Si falla cualquiera de las 7
> No envies el reporte. Investiga mas, o descartalo.

---

## Errores que te bajan el rating

| Error | Consecuencia |
|-------|--------------|
| Falsos positivos | Pierdes credibilidad |
| No revisar duplicados | Reporte cerrado, sin paga |
| PII real en PoC | Puede ser expulsion del programa |
| Lenguaje agresivo o exigente | Mal review, baneo |
| Reporte incompleto | Triage lo rechaza, pierdes tiempo |
| Scope violado | Baneo permanente de la plataforma |

---

## Recursos para estudiar reportes

| Fuente | Link |
|--------|------|
| HackerOne disclosed | hackerone.com/hacktivity |
| Bugcrowd disclosed | bugcrowd.com/bugcrowd-results |
| Report samples | PentesterLand, infosecwriteups.com |
| Writeups en Medium | medium.com/tag/bug-bounty |

> [!TIP] Aprende de los mejores
> Lee reportes reales de HackerOne disclosed. Fijate en: titulo, estructura, payload, screenshots, y como el triage respondio.

---

## Ver tambien

- [[SEGURIDAD/Roadmap Bug Bounty]]
- [[SEGURIDAD/Vulnerabilidades Web]]
- [[SEGURIDAD/Hacking Activo]]
- [[SEGURIDAD/00 - INDICE]]
