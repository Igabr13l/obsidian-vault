---
title: "Sistemas Multi-Agente (MAS)"
type: note
status: active
tags:
  - ia
  - agentes
  - arquitectura
  - mas
aliases:
  - Multi-Agent Systems
  - Enjambres de IA
created: 2026-03-24
updated: 2026-03-24
source: Apuntes sobre diseño avanzado de IA
---

# Sistemas Multi-Agente (MAS)

> [!INFO] Concepto
> Un Sistema Multi-Agente (MAS por sus siglas en inglés) es una arquitectura donde múltiples agentes de Inteligencia Artificial colaboran, debaten o compiten para resolver un problema complejo.

Si un Agente individual es un "empleado" con una serie de [[Skills (Habilidades) en IA]], un Sistema Multi-Agente es el equivalente a **una empresa entera trabajando junta**.

## ¿Por qué usar múltiples agentes? (Divide y Vencerás)

A pesar de lo potentes que son los LLMs, tienen un problema fundamental: **la degradación del prompt**. 
Si le pedís a un solo modelo que sea el Programador, el Tester, el Analista de Negocios y el Diseñador Gráfico, todo al mismo tiempo en el mismo prompt, el modelo se confunde, pierde foco y empieza a alucinar.

> [!TIP] La Solución Multi-Agente
> En lugar de un "Super Agente" que hace todo mal, creamos varios "Micro Agentes" que hacen una sola cosa perfecta, y los ponemos a conversar entre ellos.

## Roles Típicos en un MAS

Un diseño clásico (popularizado por frameworks como Microsoft AutoGen o CrewAI) incluye:

1. **El Manager (Orquestador):** Habla con el usuario, entiende el objetivo global y divide las tareas entre los demás agentes.
2. **El Especialista (Creador):** Ejecuta la tarea (ej. escribe el código o redacta el artículo). Tiene acceso a las *Tools* relevantes.
3. **El Crítico (Revisor):** Toma el trabajo del Creador, lo analiza buscando fallos y se lo devuelve con comentarios. *(Ver patrón Evaluador-Optimizador en [[Workflows (Flujos de Trabajo) en IA]])*.

## Patrones de Comunicación

¿Cómo hablan los agentes entre sí?

- **Secuencial (Cadena de montaje):** El Agente A termina su parte y se la pasa al Agente B, quien se la pasa al C. (Ej: Investigador -> Redactor -> Traductor).
- **Jerárquico:** Un Agente Manager delega a los "empleados" y consolida los resultados.
- **Debate Conjunto:** Varios agentes con distintas "personalidades" o *System Prompts* debaten sobre una idea hasta llegar a un consenso. Esto reduce drásticamente las alucinaciones.

---

## Ver también

- [[Arquitectura Cognitiva de Agentes]]
- [[Workflows (Flujos de Trabajo) en IA]]
- [[Evolución Bots, Copilots y Agentes]]
