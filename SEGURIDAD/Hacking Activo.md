---
title: "Hacking Activo"
type: note
status: active
tags:
  - seguridad
  - hacking
  - explotacion
  - bugbounty
aliases:
  - Active Exploitation
  - Cadenas de ataque
created: 2026-06-13
updated: 2026-06-13
---

# Hacking Activo

> [!INFO] Fuente
> Tecnicas de explotacion activa, encadenamiento de vulnerabilidades, y post-explotacion en bug bounty.

---

## Account Takeover (ATO)

### Vectores comunes

| Vector | Descripcion |
|--------|-------------|
| Password Reset Poisoning | Manipular el token de reset via Host header injection |
| OAuth Misconfiguration | Falta de state parameter, redirect URI manipulation |
| JWT Attacks | None algorithm, weak secret, alg confusion |
| Session Fixation | Forzar una sesion conocida en la victima |
| 2FA Bypass | Rate limiting, direct response manipulation, backup codes |
| CSRF + IDOR | Cambiar email/password de otro usuario via CSRF |

### Flujo tipico

```
1. Encontrar endpoint de cambio de email sin CSRF token
2. Crear pagina maliciosa que hace POST al endpoint
3. Victima visita la pagina → su email se cambia
4. Atacante hace "forgot password" → toma control total
```

---

## API Hacking

### GraphQL

```graphql
# Introspection (a menudo olvidada)
query {
  __schema {
    types { name fields { name } }
  }
}

# Mass assignment
mutation {
  createUser(input: {name: "test", role: "admin"}) { id }
}
```

### Common API flaws

| Flaw | Prueba |
|------|--------|
| Rate limiting ausente | Enviar muchas requests seguidas |
| Mass assignment | Agregar campos extra inesperados |
| IDOR en UUIDs | Probar UUIDs de otros usuarios |
| No content-type validation | Enviar datos como XML en vez de JSON |
| Verb tampering | Probar PATCH en vez de PUT, o DELETE en vez de POST |

---

## Server-Side Request Forgery (SSRF) Avanzado

### Bypass de allowlists

| Tecnica | Ejemplo |
|---------|---------|
| DNS rebinding | `1e100.net` resuelve a IP interna |
| URL parsing confusion | `http://google.com@127.0.0.1` |
| IPv6 loopback | `http://[::1]:8080/` |
| DNS con subdominio controlado | `attacker.com → 127.0.0.1` |
| Redirect | Usar un redirector abierto que apunte a 127.0.0.1 |

### Cadenas con SSRF

```
SSRF → http://169.254.169.254/ → AWS credentials → 
aws s3 ls s3://target-bucket → Data leak masivo

SSRF → http://127.0.0.1:9200/ → Elasticsearch sin auth → 
Todos los datos internos expuestos

SSRF → http://127.0.0.1:3000/ → Servicio interno vulnerable → RCE
```

---

## Request Smuggling

### Tipos

| Tipo | Sintoma |
|------|---------|
| CL.TE | Proxy usa Content-Length, backend usa Transfer-Encoding |
| TE.CL | Proxy usa Transfer-Encoding, backend usa Content-Length |
| TE.TE | Ambos usan TE, pero uno se puede ofuscar |

### Deteccion

```http
POST / HTTP/1.1
Host: target.com
Content-Length: 6
Transfer-Encoding: chunked

0

X
```

> [!WARNING] Impacto
> Request smuggling puede derivar en: XSS reflejado en otros usuarios, cache poisoning, bypass de restricciones de WAF.

---

## Subdomain Takeover

### Checklist

```
1. Encontrar CNAME apuntando a servicio externo (AWS S3, Heroku, GitHub Pages, etc.)
2. Verificar que el recurso externo NO existe (404/NXDOMAIN)
3. Reclamar el recurso en el servicio externo
4. Subir contenido → tienes control de subdominio objetivo
```

### Servicios comunes

| Servicio | Indicador |
|----------|-----------|
| AWS S3 | `NoSuchBucket` |
| Heroku | `There's nothing here, yet.` |
| GitHub Pages | `404: Not Found` |
| Shopify | `Sorry, this shop is currently unavailable.` |
| Azure | `The specified resource does not exist` |

---

## Herramientas de explotacion

| Herramienta | Uso |
|-------------|-----|
| nuclei | Escaneo automatico basado en templates |
| metasploit | Post-explotacion, payloads |
| Burp Suite Pro | Proxy + Repeater + Intruder + Scanner |
| sqlmap | Automatizacion de SQLi |
| xsstrike | Deteccion avanzada de XSS |
| jwt_tool | Analisis y manipulacion de JWT |
| smuggler | Deteccion de HTTP request smuggling |

---

## Ver tambien

- [[SEGURIDAD/Roadmap Bug Bounty]]
- [[SEGURIDAD/Vulnerabilidades Web]]
- [[SEGURIDAD/Reconocimiento]]
- [[SEGURIDAD/Reportes]]
- [[SEGURIDAD/00 - INDICE]]
