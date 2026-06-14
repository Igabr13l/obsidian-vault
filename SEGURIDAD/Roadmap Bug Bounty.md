---
title: "Roadmap Bug Bounty"
type: note
status: active
tags:
  - seguridad
  - bugbounty
  - roadmap
  - aprendizaje
aliases:
  - Roadmap Ciberseguridad
  - Plan de estudios bug bounty
created: 2026-06-13
updated: 2026-06-13
---

# Roadmap Bug Bounty

> [!INFO] Fuente
> Basado en rutas de aprendizaje de PortSwigger, HackTheBox, Bugcrowd University, y experiencia de investigadores como STOK, InsiderPhD, y Jhaddix.

---

## Fase 0: Fundamentos (1-2 meses)

### Redes y Protocolos

| Tema | Recurso |
|------|---------|
| Modelo OSI / TCP-IP | Professor Messer, NetworkChuck |
| HTTP/HTTPS (metodos, headers, cookies, sesiones) | MDN Web Docs |
| DNS (tipos de registros, resolucion) | DNS in Detail (TryHackMe) |
| Linux basico (terminal, permisos, grep/awk) | Linux Journey, OverTheWire Bandit |

### Web Basics

| Tema | Recurso |
|------|---------|
| HTML, CSS, JavaScript (nociones) | freeCodeCamp |
| APIs REST, GraphQL, WebSockets | Documentacion abierta |
| Auth (cookies, JWT, OAuth, SAML) | PortSwigger Academy |

### Herramientas iniciales

| Herramienta | Uso |
|-------------|-----|
| Burp Suite Community | Proxy de intercepcion |
| curl / httpie | Peticiones desde terminal |
| DevTools (Chrome/Firefox) | Inspeccion de trafico |
| nmap | Escaneo de puertos |

> [!TIP] Meta de la fase
> Poder entender una peticion HTTP completa (request/response), saber que hace cada parte, y poder modificarla con Burp.

---

## Fase 1: Reconocimiento y OSINT (2-3 meses)

Ver nota dedicada: [[SEGURIDAD/Reconocimiento]]

### Pasiva (sin tocar el target)

| Tecnica | Herramientas |
|---------|--------------|
| Subdominios | subfinder, amass, crt.sh, Sublist3r |
| URLs historicas | gau, waybackurls, katana |
| Parametros | paramspider, arjun |
| Tecnologias | Wappalyzer, whatweb |
| WHois / ASN | whois, bgp.he.net |
| Google Dorks | Google Hacking Database |
| GitHub dorks | gitdorker, trufflehog |

### Activa (con precaucion)

| Tecnica | Herramientas |
|---------|--------------|
| Escaneo de puertos | nmap, masscan, rustscan |
| Fuzzing de directorios | ffuf, dirsearch, gobuster |
| Crawling | katana, gospider, burp spider |
| Screenshots | aquatone, gowitness |

> [!WARNING] Scope
> Nunca escanees ni toques un target sin autorizacion explicita. Usa siempre entornos legales (HackTheBox, PentesterLab, programas de bug bounty con scope claro).

---

## Fase 2: Vulnerabilidades Web (3-6 meses)

Ver nota dedicada: [[SEGURIDAD/Vulnerabilidades Web]]

### Orden recomendado de aprendizaje

| Prioridad | Vulnerabilidad | Recurso principal |
|-----------|---------------|-------------------|
| 1 | XSS (Reflejado, Almacenado, DOM) | PortSwigger XSS |
| 2 | SQL Injection | PortSwigger SQLi |
| 3 | CSRF | PortSwigger CSRF |
| 4 | SSRF | PortSwigger SSRF |
| 5 | IDOR / BAC | PortSwigger Access Control |
| 6 | File Upload / LFI / RFI | PayloadsAllTheThings |
| 7 | SSTI | PortSwigger SSTI |
| 8 | XXE | PortSwigger XXE |
| 9 | Open Redirect | PortSwigger |
| 10 | Race Conditions | PortSwigger |

### Plataformas de practica

| Plataforma | Tipo | Costo |
|------------|------|-------|
| PortSwigger Web Security Academy | Labs gratuitos | Gratis |
| HackTheBox (Machines + Academy) | Labs + cursos | Gratis/Pago |
| TryHackMe | Labs guiados | Gratis/Pago |
| PentesterLab | Labs progresivos | Pago |
| OWASP Juice Shop | App vulnerable | Gratis |
| DVWA | App vulnerable | Gratis |
| HackTheBox | Labs desafiantes | Pago |

> [!TIP] Estrategia
> Por cada vulnerabilidad: 1) lee la teoria, 2) hace los labs de PortSwigger, 3) practica en Juice Shop/DVWA, 4) busca reportes reales en HackerOne disclosed.

---

## Fase 3: Hacking Activo y Cadenas de Ataque (3-6 meses)

Ver nota dedicada: [[SEGURIDAD/Hacking Activo]]

### Tecnicas avanzadas

| Tecnica | Descripcion |
|---------|-------------|
| Auth bypass | MFA bypass, OAuth misconfig, JWT attacks |
| Business Logic | Flujos alternativos, race conditions, order manipulation |
| API Hacking | GraphQL introspection, mass assignment, rate limiting |
| Cache poisoning/deception | Web cache poisoning, CDN confusion |
| HTTP Request Smuggling | CL.TE, TE.CL, TE.TE |
| Prototype Pollution | Client-side, server-side |
| Subdomain Takeover | DNS check, service detection |

### Cadenas de ataque comunes

```
Recon → Encontrar subdominio olvidado → Subdomain takeover → 
Ejecutar XSS → Robar cookies → Account takeover
  
Encontrar endpoint con IDOR → Escalar a admin → PII leak masivo

SSRF → Leer metadata cloud (AWS IMDS) → Access keys → Full compromise
```

> [!IMPORTANT] Reportes reales
> Estudia reportes publicados en HackerOne disclosed y Bugcrowd. Analiza el payload, la cadena de ataque, y como lo reportaron.

---

## Fase 4: Reportes y Comunicacion (continuo)

Ver nota dedicada: [[SEGURIDAD/Reportes]]

### Estructura de un reporte

1. **Resumen** — Que y donde
2. **Impacto** — Por que importa
3. **Pasos para reproducir** — Paso a paso con screenshots
4. **Payload / PoC** — Prueba de concepto clara
5. **Impacto potencial** — Lo peor que podria pasar
6. **Remediacion sugerida** — Como arreglarlo

> [!WARNING] Errores comunes
> - Reportar sin probar bien (falsos positivos)
> - No respetar el VRT (Vulnerability Rating Taxonomy)
> - Duplicados (revisa triage previamente)
> - Datos sensibles en el reporte (nunca incluyas PII real)

---

## Timeline estimado

| Fase | Duracion | Nivel al finalizar |
|------|----------|-------------------|
| 0 | 1-2 meses | Completas los Bandit de OverTheWire, conoces HTTP |
| 1 | 2-3 meses | Podes hacer recon completo de un target |
| 2 | 3-6 meses | Resolves PortSwigger Practitioner, encuentras XSS/SQLi reales |
| 3 | 3-6 meses | Encadenas vulnerabilidades, entiendes ATO y SSRF->cloud |
| 4 | Continuo | Reportas claramente, recibes triages |

### Total estimado: 12-18 meses para ser competitivo en bug bounty

> [!NOTE] Ritmo personal
> Esto no es una carrera. Muchos investigadores tardan 2+ años en recibir su primer bounty grande. Lo importante es la consistencia, no la velocidad.

---

## Recursos recomendados

### Libros

- *Web Application Hacker's Handbook* — Stuttard & Pinto
- *Real-World Bug Hunting* — Peter Yaworski
- *The Browser Hacker's Handbook* — Wade et al.

### Creadores / Canales

| Nombre | Enfoque |
|--------|---------|
| STOK (NahamSec) | Bug bounty general, mindset |
| InsiderPhD | Metalearning, enfoque academico |
| Jhaddix | Recon, automatizacion |
| Farah Hawa | XSS, web hacking |
| Rana Khalil | PortSwigger labs walkthrough |
| IppSec | HackTheBox walkthroughs |

### Repositorios clave

- PayloadsAllTheThings — Payloads por tipo de vulnerabilidad
- SecLists — Wordlists de todo tipo
- HackerOne disclosed — Reportes reales
- ProjectDiscovery — Suite de herramientas (nuclei, httpx, subfinder)

---

## Ver tambien

- [[SEGURIDAD/00 - INDICE]]
- [[SEGURIDAD/Reconocimiento]]
- [[SEGURIDAD/Vulnerabilidades Web]]
- [[SEGURIDAD/Hacking Activo]]
- [[SEGURIDAD/Reportes]]
- [[ESTUDIO/00 - INDICE]]
