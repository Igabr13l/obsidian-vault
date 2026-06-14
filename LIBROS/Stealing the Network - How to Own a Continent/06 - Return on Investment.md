---
title: "Capitulo 6 - Return on Investment"
type: note
status: active
tags:
  - libros
  - seguridad
  - stealing-the-network
  - rootkits
  - sniffing
aliases:
  - Return on Investment
created: 2026-06-13
updated: 2026-06-13
---

# Capitulo 6 — Return on Investment

> [!INFO] Ficha
> - **Páginas:** 38
> - **Tema:** Sniffing, rootkits y monetizacion

---

## Trama

El capitulo explora la **monetizacion del acceso comprometido**. Una vez que tienes control de un sistema, como sacarle el maximo provecho economico antes de que lo descubran.

## Temas tecnicos

| Tecnica | Detalle |
|---------|---------|
| **Packet sniffing** | Capturar trafico de red para robar credenciales y datos |
| **Rootkits** | Ocultar la presencia del atacante en el sistema |
| **Data exfiltration** | Extraer datos valiosos sin ser detectado |
| **Lateral movement** | Usar un sistema comprometido para saltar a otros |
| **ROI del ataque** | Como decidir que datos robar y como venderlos |

## Tecnica destacada: Rootkits

El capitulo detalla como instalar rootkits que:
- Oculten procesos y archivos del atacante
- Provean backdoors persistentes
- Evadan deteccion por antivirus
- Limpien logs y huellas

> [!NOTE] Evolucion
> Los rootkits de 2004 operaban a nivel de kernel. Hoy el malware fileless y los rootkits de firmware son la evolucion natural.

## Leccion clave

Para un atacante, el **tiempo de permanencia** (dwell time) es todo. Cuanto mas tiempo pase sin ser detectado, mas datos puede robar. El rootkit es la herramienta que hace posible esa permanencia.

## Ver tambien

- [[LIBROS/Stealing the Network - How to Own a Continent/00 - INDICE]]
