---
title: "Capitulo 9 - Automatic Terror Machine"
type: note
status: active
tags:
  - libros
  - seguridad
  - stealing-the-network
  - malware
aliases:
  - Automatic Terror Machine
created: 2026-06-13
updated: 2026-06-13
---

# Capitulo 9 — Automatic Terror Machine

> [!INFO] Ficha
> - **Páginas:** 22
> - **Tema:** Malware y armas automaticas

---

## Trama

El capitulo mas controvertido del libro. Describe un **malware autonomo** diseñado para propagarse sin control, causando dano masivo en infraestructura critica antes de ser detenido. La falta de claridad sobre la intencion del ataque hace el capitulo "perplexing" segun un reviewer.

## Temas tecnicos

| Tecnica | Detalle |
|---------|---------|
| **Gusano autonomo** | Malware que se propaga automaticamente |
| **Propagacion dirigida** | Busqueda de objetivos especificos (SCADA, ICS) |
| **Payload destructivo** | Dano fisico a infraestructura |
| **Command & Control** | Comunicacion con servidores de mANDO |

## El problema del control

El capitulo muestra las dificultades de dirigir un ataque automatico:
- Como evitar que el malware ataque objetivos equivocados
- Como detenerlo cuando ya se descontrolo
- El riesgo de "blowback" (que el ataque vuelva contra su creador)

> [!WARNING] Critica del reviewer
> "There are the usual unresolved problems with directing attacks and limiting spread. The lack of particulars on the intent of the attack makes the chapter quite perplexing."

## Leccion clave

El malware autonomo es un **arma de doble filo**. Una vez liberado, el creador pierde control. Stuxnet (2010) demostro que es posible, pero requirio anos de desarrollo y recursos de un estado-nacion.

## Ver tambien

- [[LIBROS/Stealing the Network - How to Own a Continent/00 - INDICE]]
