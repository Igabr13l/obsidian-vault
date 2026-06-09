---
title: "Prompt Engineering"
type: reference
status: active
tags:
  - llm
  - ia
  - glosario
  - prompt-engineering
aliases:
  - Ingeniería de prompts
created: 2026-03-08
updated: 2026-03-08
---

# Prompt Engineering

> [!DEFINITION] Definición
> Diseño intencional de instrucciones, contexto, restricciones y ejemplos para guiar la salida de un modelo hacia un resultado util y controlable.

---

## Concepto principal

Prompt engineering es la practica de escribir mejores instrucciones para que un LLM entienda:

- que tarea debe resolver
- para quien lo hace
- que formato debe usar
- que limites o criterios debe respetar

No se trata de "palabras magicas", sino de reducir ambiguedad.

---

## Componentes habituales

### Contexto

Define el rol, el dominio o la situacion en la que debe responder el modelo.

### Especificidad

Mientras mas concreto sea el pedido, menor margen hay para una respuesta generica.

### Formato

Indicar si queres JSON, Markdown, tabla o bullets vuelve la salida mas consistente.

### Restricciones

Pautas como longitud maxima, tono o cosas a evitar ayudan a mantener el resultado dentro del objetivo.

### Ejemplos

Dar uno o mas ejemplos de entrada/salida mejora mucho la imitacion del patron esperado.

---

## Tecnicas relacionadas

| Tecnica | Para que sirve |
|---------|----------------|
| [[Chain of Thought]] | Pedir razonamiento paso a paso |
| Few-shot prompting | Mostrar ejemplos para fijar formato o tono |
| Output structuring | Forzar una estructura reutilizable |
| Constraint prompting | Limitar longitud, estilo o alcance |

---

## Aplicacion practica

En agentes de codigo, un buen prompt puede funcionar como una mini especificacion:

- define el objetivo
- ordena prioridades
- reduce cambios innecesarios
- mejora la calidad del primer intento

> [!EXAMPLE] Ejemplo
> En vez de pedir `revisa este codigo`, conviene pedir: `revisa este codigo priorizando seguridad, luego performance y por ultimo legibilidad; responde con Issue, Severity y Fix`.

---

## Nota relacionada

Una buena expansion de este concepto esta en [[Como escribir prompts para shippear 10x mas rapido]].

---

## Ver también

- [[Chain of Thought|Cadena de Pensamiento (Chain of Thought)]]
- [[Como escribir prompts para shippear 10x mas rapido]]
- [[Glosario LLM]]
