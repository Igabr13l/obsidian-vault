---
title: "Fundamentos de Redes para Hacking"
type: note
status: active
tags:
  - seguridad
  - redes
  - fundamentos
  - aprendizaje
aliases:
  - Redes para hacking
  - Network fundamentals
created: 2026-06-13
updated: 2026-06-13
---

# Fundamentos de Redes para Hacking

> [!INFO] Fuente
> Basado en Professor Messer, NetworkChuck, y experiencia practica en TryHackMe/HackTheBox.

---

## Modelo OSI (7 capas)

| Capa | Nombre | Protocolos | Que atacar |
|------|--------|------------|------------|
| 7 | Aplicacion | HTTP, FTP, SMTP, DNS | Todo lo web (XSS, SQLi, SSRF) |
| 6 | Presentacion | SSL/TLS, JPEG, ASCII | SSL stripping, downgrade attacks |
| 5 | Sesion | NetBIOS, RPC | Session hijacking |
| 4 | Transporte | TCP, UDP | Port scanning, SYN flood |
| 3 | Red | IP, ICMP, ARP | IP spoofing, ARP poisoning |
| 2 | Enlace | Ethernet, WiFi (802.11) | MAC spoofing, ARP spoofing |
| 1 | Fisica | Cables, ondas de radio | Hacking fisico, jamming |

> [!TIP] Relevancia para bug bounty
> En bug bounty web el 80% del tiempo operas en capa 7 (HTTP). Para CTF/HTB necesitas capa 3-4. Para red team, capa 2.

---

## TCP vs UDP

### Flags TCP importantes

| Flag | Significado | Uso en hacking |
|------|-------------|----------------|
| SYN | Iniciar conexion | Escaneo -sS (SYN scan) |
| ACK | Confirmar recepcion | Escaneo -sA (firewall mapping) |
| FIN | Cerrar conexion | Escaneo -sF (evasion) |
| RST | Reiniciar conexion | Firewalls envian RST para bloquear |
| PSH | Enviar datos inmediato | Exploits, data exfiltration |
| URG | Datos urgentes | Raramente usado, evasivo |

### Handshake TCP

```
Cliente → SYN → Servidor
Cliente ← SYN-ACK ← Servidor
Cliente → ACK → Servidor
```

> [!NOTE] Evasion de firewalls
> Algunos firewalls solo bloquean SYN. Usar escaneos FIN (-sF) o NULL (-sN) puede evadirlos.

---

## DNS

### Tipos de registros

| Tipo | Significado | Uso en hacking |
|------|-------------|----------------|
| A | IPv4 | Mapeo directo |
| AAAA | IPv6 | Mapeo IPv6 |
| CNAME | Alias a otro dominio | Subdomain takeover |
| MX | Mail server | OSINT, SPF/DMARC checks |
| NS | Name server | Zone transfer attacks |
| TXT | Texto arbitrario | SPF, DKIM, DMARC, verificaciones |
| SOA | Start of Authority | Info del dominio |

### Ataques comunes a DNS

| Ataque | Descripcion |
|--------|-------------|
| Zone Transfer | Si AXFR esta habilitado, te llevas todos los registros del dominio |
| DNS Rebinding | Usar dominio propio que cambia resolucion para bypassear SSRF |
| Subdomain Takeover | CNAME a servicio muerto, lo reclamás |
| DNS Cache Poisoning | Envenenar cache de resolvers |

```bash
# Zone transfer
dig axfr @ns1.target.com target.com

# Enumeracion de subdominios
dnsrecon -d target.com
dnsenum target.com
```

---

## HTTP/HTTPS

> Ver nota especifica: [[SEGURIDAD/HTTP Profundo]]

### Metodos clave

| Metodo | Uso | Notas de seguridad |
|--------|-----|-------------------|
| GET | Obtener recurso | Parametros en URL (logeados) |
| POST | Enviar datos | Body, CSRF aplica |
| PUT | Subir recurso | Si esta habilitado, podes subir webshells |
| DELETE | Borrar recurso | Si esta habilitado, podes borrar todo |
| PATCH | Modificar parcial | A veces no tiene auth |
| OPTIONS | Ver metodos disponibles | Te dice que metodos acepta el servidor |

### Codigos de estado

```
1xx → Informacion (continuar, switching protocols)
2xx → Exito (200 OK, 201 Created, 204 No Content)
3xx → Redireccion (301 moved, 302 found, 307 redirect)
4xx → Error cliente (400 bad request, 401 unauth, 403 forbidden, 404 not found)
5xx → Error servidor (500 internal, 502 bad gateway, 503 unavailable)
```

> [!TIP] Lo que importa en bug bounty
> - `403` puede ser falso (bypasseable con headers X-Forwarded-For)
> - `200` en paginas que deberian ser `404` indica algo raro
> - `500` en endpoints con parametros raros = probable SQLi/SSTI

---

## Headers HTTP clave para seguridad

### Request Headers

| Header | Que hace | Relevancia |
|--------|----------|------------|
| `Host` | Dominio destino | Host header injection |
| `Cookie` | Sesion | Session hijacking, XSS |
| `Authorization` | Credenciales | JWT, Basic auth |
| `Origin` | Origen de la request | CORS |
| `Referer` | Pagina anterior | CSRF checks |
| `X-Forwarded-For` | IP real del cliente | Bypass de restricciones por IP |
| `Content-Type` | Tipo de contenido | Content-type confusion |
| `User-Agent` | Browser/cliente | Fingerprinting |

### Response Headers de seguridad

| Header | Que hace | Si falta... |
|--------|----------|-------------|
| `Content-Security-Policy` | Controla que recursos cargar | XSS mas facil |
| `X-Frame-Options` | Previene clickjacking | Podes iframear el sitio |
| `X-Content-Type-Options: nosniff` | Previene MIME sniffing | File upload puede ejecutar JS |
| `Strict-Transport-Security` | Fuerza HTTPS | SSL stripping |
| `Set-Cookie: HttpOnly` | Cookie no accesible via JS | XSS puede robar cookies |
| `Set-Cookie: Secure` | Cookie solo via HTTPS | Cookie enviada en HTTP |
| `Set-Cookie: SameSite` | Previene CSRF | CSRF mas facil |

---

## Herramientas de red esenciales

```bash
# Analisis de conectividad
ping target.com          # Latencia, perdida de paquetes
traceroute target.com    # Ruta de paquetes
mtr target.com           # traceroute + ping en vivo

# Escaneo de puertos
nmap -sS -sV -p- target.com    # SYN scan completo
nmap -sU -p 53,161 target.com  # UDP scan (lento, solo puertos clave)

# Captura de trafico
tcpdump -i eth0 port 80        # Capturar HTTP
tcpdump -i eth0 -w captura.pcap # Guardar para analisis

# DNS
dig target.com ANY
nslookup target.com
host -t mx target.com

# HTTP
curl -v https://target.com                 # Request completa
curl -X POST -d "user=admin" target.com     # POST data
curl -H "X-Forwarded-For: 127.0.0.1" url   # Header spoofing
```

---

## Conceptos de red para hacking

| Concepto | Explicacion | Uso |
|----------|-------------|-----|
| NAT | Traduccion de IP privada a publica | Dificulta tracking, complica pivoting |
| Proxy | Intermediario que reenvia requests | Burp Suite, mitmproxy |
| VPN | Tunel cifrado | Acceso a redes internas |
| CDN | Red de distribucion de contenido | Cache poisoning, IP origin bypass |
| Load Balancer | Distribuye trafico entre servidores | Request smuggling, sticky sessions |
| WAF | Firewall de aplicacion web | Bypass, deteccion de reglas |

> [!TIP] Antes de atacar, entendé la infraestructura
> - Si hay WAF, vas a necesitar bypasses
> - Si hay CDN, busca la IP real (Shodan, Censys, historical DNS)
> - Si hay load balancer, proba request smuggling

---

## Ver tambien

- [[SEGURIDAD/HTTP Profundo]]
- [[SEGURIDAD/Linux para Hacking]]
- [[SEGURIDAD/Roadmap Bug Bounty]]
- [[SEGURIDAD/00 - INDICE]]
