---
title: "OSINT Profundo"
type: note
status: active
tags:
  - seguridad
  - osint
  - recon
  - social-engineering
aliases:
  - Deep OSINT
  - Investigacion OSINT
  - Social Engineering OSINT
created: 2026-06-13
updated: 2026-06-13
---

# OSINT Profundo

> [!INFO] Fuente
> Basado en metodologias de OSINT, The OSINTion, IntelTechniques, y reportes reales.

---

## Email OSINT

### Verificar existencia

```bash
# Verificar si un email existe en un servicio
hunter.io/search?domain=target.com
verify-email.org

# Breach data
haveibeenpwned.com/account/{email}
dehashed.com
leak-check.net

# Google dork
"email@target.com" site:github.com
"email@target.com" site:pastebin.com
```

### Enumeracion de empleados

| Herramienta | Uso |
|-------------|-----|
| Hunter.io | Encontrar emails por dominio |
| RocketReach | Emails + perfiles de LinkedIn |
| Skymem | Base de datos de emails publicos |
| Phonebook.cz | Emails, subdominios, URLs |
| Clearbit | API para enrichment de emails |

### Verificacion de formato

```
# Formatos comunes en empresas
firstname@target.com
firstname.lastname@target.com
firstinitial_lastname@target.com
f.lastname@target.com
```

---

## Persona Tracking

### LinkedIn OSINT

```bash
# Herramientas
linkedin2username  # Generar usernames desde empleados de LinkedIn
crosslinked        # Buscar el mismo username en otras redes

# Tecnica manual
1. Buscar empleados de target.com en LinkedIn
2. Identificar roles: ing., security, devops, C-level
3. Extraer tecnologias que mencionan en sus perfiles
4. Ver educacion, empleos previos, certificaciones

# Lo que te importa:
- Que stack tecnologico usan (lo mencionan los devs)
- Quien es el CISO/security (target de phishing)
- Ex-empleados (posible acceso residual, resentimiento)
```

### Redes sociales

```bash
# Busqueda cross-platform
sherlock USERNAME
maigret USERNAME
holehe EMAIL
social-analyzer USERNAME

# what3words para ubicaciones
```

### GitHub OSINT

```bash
# Buscar en repositorios de la empresa
org:target "password"
org:target "api_key"
org:target "aws_secret"
org:target ".env"
org:target "-----BEGIN RSA PRIVATE KEY-----"

# Buscar empleados
gitdorker -q target.com -d dictionaries/dork_dict.txt
trufflehog --regex --entropy=True https://github.com/target/repo.git

# Commit history (a veces borran secretos pero quedan en git history)
git log --all --full-history -- **/wp-config.php
git log -S "password" --all
```

---

## Data Breaches

### Fuentes

| Fuente | Tipo | Notas |
|--------|------|-------|
| Dehashed | Pago | Base mas completa, busqueda por email/user/ip |
| Have I Been Pwned | Gratis | Solo verifica si un email esta en breaches |
| LeakCheck | Gratis/Pago | Busqueda por email, user, password |
| Snusbase | Pago | Combinacion de muchas breaches |
| IntelX | Pago | Breaches + pastebin + dark web |
| Scylla.so | Gratis | Base de breaches (antigua pero util) |

> [!WARNING] Etica
> Usar data de breaches para acceder a cuentas es ilegal. Usalo solo para:
> - Verificar si tus propias credenciales estan filtradas
> - Demostrar riesgo a un cliente (reuse de passwords)

### Tecnica: Password spraying

Si tenes emails de empleados y sabes que hubo un breach previo:

1. Obtener lista de emails validos
2. Probar las TOP 10 passwords filtradas globalmente
3. Bajo volumen (1 intento por hora por cuenta) para evitar lockout

---

## Social Engineering OSINT

### Construccion de pretexto

1. **Empresa**: target.com
2. **Rol**: Soporte tecnico de un vendor (Microsoft, AWS, Cloudflare)
3. **Canal**: Phone, LinkedIn DM, Email
4. **Pretexto**: "Estamos auditando licencias, necesito confirmar su version de X"
5. **Info buscada**: Stack tecnologico, versiones, nombre de admins

### OSINT para spear phishing

```yaml
Target: Juan Perez, DevOps en target.com
Datos:
  - Trabaja ahi desde 2023
  - Menciono en LinkedIn que usan Kubernetes
  - Su email: j.perez@target.com
  - Tiene GitHub con proyectos personales
  - Sigue a ciertos influencers tech

Aproximacion:
  "Hola Juan, vi tu proyecto en GitHub sobre Kubernetes.
  Estamos organizando una meetup virtual sobre K8s security.
  Te paso el link para registrarte: [malicious link]"
```

---

## Tecnicas avanzadas

### Wayback Machine

```bash
# Ver versiones historicas del sitio
web.archive.org/web/*/target.com

# Buscar archivos especificos
curl "https://web.archive.org/cdx/search/cdx?url=target.com&output=json" | jq '.[] | select(.[2] | test("\\.js$|\\.php$|\\.env$"))'

# Lo que buscas:
- Endpoints que ya no existen pero estan documentados
- Version anterior de la API (sin auth)
- Archivos .env, .git, wp-config expuestos historicamente
```

### Certificate Transparency

```bash
# crt.sh con formato JSON
curl -s "https://crt.sh/?q=%25.target.com&output=json" | jq -r '.[].name_value' | sort -u

# Buscar wildcards
curl -s "https://crt.sh/?q=%25.target.com&output=json" | jq -r '.[].name_value' | grep -E '^\*\.'

# Historicamente
curl -s "https://crt.sh/?q=%25.target.com&output=json" | jq -r '.[] | "\(.name_value) \(.not_after)"' | sort -k2
```

### Shodan / Censys

```bash
# Buscar IPs reales detras de Cloudflare
https://www.shodan.io/search?query=http.title%3A%22target+title%22
https://censys.io/ipv4?q=services.http.response.html_title%3A%22target+title%22

# Buscar por certificado SSL
https://www.shodan.io/search?query=ssl.cert.subject.cn%3Atarget.com

# Buscar tecnologias especificas
https://www.shodan.io/search?query=server%3A%22Apache%22+country%3A%22AR%22
```

---

## Flujo OSINT completo

```
1. DOMAIN → Subdominios (crt.sh, subfinder) → IPs (dig)
2. IPs → Shodan/Censys → Puertos abiertos, tecnologias
3. TECHNOLOGIES → Based on versiones → CVEs conocidos
4. EMAILS → Hunter.io → Emails de empleados
5. EMPLOYEES → LinkedIn → Roles, stack tecnologico
6. BREACHES → Dehashed → Passwords filtradas
7. GITHUB → Busqueda de secretos → Access keys
8. SOCIAL → Pretexto → Spear phishing
```

---

## Herramientas OSINT

| Herramienta | Uso | Instalacion |
|-------------|-----|-------------|
| Sherlock | Buscar username en redes | `pip install sherlock` |
| Maigret | Mas exhaustivo que sherlock | `pip install maigret` |
| Holehe | Verificar si email esta en servicios | `pip install holehe` |
| GitDorker | Buscar secretos en GitHub | `git clone` |
| TruffleHog | Escaneo de secretos en git | `pip install trufflehog` |
| TheHarvester | Emails, subdominios, IPs | `apt install theharvester` |
| Recon-ng | Framework OSINT completo | `git clone` |

---

## Ver tambien

- [[SEGURIDAD/Reconocimiento]]
- [[SEGURIDAD/Roadmap Bug Bounty]]
- [[SEGURIDAD/Hacking Activo]]
- [[SEGURIDAD/00 - INDICE]]
