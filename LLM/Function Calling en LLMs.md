---
title: "Function Calling en LLMs"
type: note
status: active
tags:
  - ia
  - llm
  - desarrollo
  - tools
aliases:
  - Uso de herramientas
  - Tool calling
created: 2026-03-24
updated: 2026-03-24
source: Apuntes técnicos sobre interacción entre LLMs y sistemas externos
---

# Function Calling en LLMs

> [!INFO] Definición
> **Function Calling** (o Tool Calling) es la capacidad de un Modelo de Lenguaje de entender que necesita usar una herramienta externa para cumplir una tarea, y generar los parámetros exactos para usarla.

Es la pieza de tecnología exacta que permite a un LLM "salir de su caja" y convertirse en un Agente que interactúa con el mundo real.

## La Ilusión de la Acción

Hay un malentendido muy común: **Los LLMs NO ejecutan código, ni aprietan botones, ni navegan por internet por sí mismos.** Un LLM solo entra texto y escupe texto.

> [!WARNING] El Truco de Magia
> Cuando le pedís a un Agente "Buscame el clima en Buenos Aires", el LLM no se conecta a internet. Lo que hace es generar un texto en un formato estructurado (generalmente un `JSON`) que dice: *"Por favor, sistema externo, ejecutá la función `get_weather` con el parámetro `city: "Buenos Aires"`"*. 
> 
> Es tu computadora (el sistema alrededor del modelo) la que realmente hace la llamada a internet y le devuelve el resultado en texto al LLM.

## El Ciclo de Vida del Function Calling

Para que un Agente use una *Tool*, ocurre este flujo invisible en milisegundos:

1. **Definición:** El programador le envía al LLM un manual explicando qué herramientas existen (ej. *"Tenés una herramienta que se llama `calc_sum(a, b)` y sirve para sumar"*).
2. **Decisión (LLM):** El usuario pide *"Cuánto es 145 + 322"*. El LLM se da cuenta de que no debe adivinar, sino usar la herramienta.
3. **Generación (LLM):** El modelo devuelve un JSON: `{ "name": "calc_sum", "args": {"a": 145, "b": 322} }`. El modelo se pausa.
4. **Ejecución (Sistema):** El framework (como LangChain o Antigravity) agarra ese JSON, ejecuta la función real de Python/JavaScript y obtiene el resultado (`467`).
5. **Inyección de Resultado (Sistema -> LLM):** El sistema le vuelve a hablar al modelo: *"La herramienta devolvió 467"*.
6. **Respuesta Final (LLM):** El modelo retoma el habla y le dice al usuario: *"La suma de 145 y 322 es 467"*.

### ¿Por qué es revolucionario?
Antes del Function Calling (introducido masivamente por OpenAI a mediados de 2023), los programadores tenían que rogarle al modelo mediante el *prompt* que respondiera en un formato limpio para poder leerlo. Hoy, los modelos están entrenados (Fine-tuned) a nivel neuronal específicamente para escupir JSONs perfectos y fiables cuando se les pide que usen una herramienta.

---

## Ver también

- [[Arquitectura Cognitiva de Agentes]]
- [[Workflows (Flujos de Trabajo) en IA]]
- [[Sistemas Multi-Agente (MAS)]]
