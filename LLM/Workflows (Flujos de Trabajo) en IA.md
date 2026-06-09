---
title: "Workflows (Flujos de Trabajo) en IA"
type: note
status: active
tags:
  - ia
  - agentes
  - automatizacion
  - arquitectura
aliases:
  - Workflows en IA
  - DAGs en Agentes
  - Patrones de Orquestación
created: 2026-03-24
updated: 2026-03-24
source: Apuntes de estudio sobre ecosistemas de agentes
---

# Workflows (Flujos de Trabajo) en IA

> [!INFO] Definición
> Un **Workflow** es la estructura lógica (una receta paso a paso o un grafo) que orquesta *cuándo* y *cómo* el LLM piensa, qué *Skills* debe usar y qué *Tools* debe ejecutar para resolver una tarea compleja sin desviarse del objetivo.

Cuando a un LLM se le pide una tarea que toma mucho tiempo o múltiples pasos lógicos, la probabilidad de que "alucine" o se olvide del objetivo inicial se acerca al 100%. Los Workflows son las **"barreras de contención" (guardrails)** que evitan esto.

## Problemas que resuelve un Workflow

1. **Alucinación en Cascada:** Si el paso 2 falla, un LLM "suelto" podría inventar un resultado para poder seguir al paso 3. Un workflow detiene el proceso y maneja el error.
2. **Ciclos Infinitos (Infinite Loops):** Los agentes autónomos suelen quedarse atrapados intentando la misma solución fallida una y otra vez. Los workflows imponen un límite de intentos (*max_retries*).
3. **Pérdida de Foco:** Divide el problema en nodos discretos. En el nodo "Planificar", el LLM no piensa en código. En el nodo "Programar", no piensa en el plan general, solo ejecuta.

## Patrones Comunes de Workflows

A nivel técnico, los workflows suelen construirse como **Máquinas de Estado** o **Grafos Dirigidos Acíclicos (DAGs)**.

### 1. Patrón Secuencial (Cadena)
El más simple. El output de un paso es el input del siguiente.
*A (Investigar) -> B (Resumir) -> C (Traducir).*

### 2. Enrutador (Router)
El agente toma una decisión inicial que bifurca el camino.
> *Usuario pide algo -> (LLM decide)*
> - *Si es código -> Enruta al Workflow de Desarrollo.*
> - *Si es texto -> Enruta al Workflow de Redacción.*

### 3. Evaluador-Optimizador (Reflection Loop)
Es el núcleo de la autonomía real de alta calidad.
1. El Agente "Creador" hace un borrador (ej. escribe una función).
2. Pasa el resultado a un Agente "Crítico" o "Test" (ej. compila el código).
3. Si el test falla, el error se envía de vuelta al "Creador" para corregirlo.
4. El ciclo (loop) se repite hasta que pasa la prueba o se alcanza el límite.

### 4. Human-in-the-Loop (HITL)
Vital en sistemas empresariales o entornos seguros como Antigravity. El workflow se pausa obligatoriamente en nodos críticos (ej. antes de borrar archivos, ejecutar un pago o hacer commit de código) y espera la confirmación explícita de un humano para avanzar.

> [!EXAMPLE] Ejemplo Completo en un Agente
> 1. (Inicio del Workflow)
> 2. Cargar **Skill** de "Escritor de Markdown".
> 3. Usar **Tool** `buscar_en_internet` sobre el tema X.
> 4. (Nodo de control) ¿Hay suficiente información? Si no, iterar. Si sí, avanzar.
> 5. Escribir borrador.
> 6. **HITL:** Pedir aprobación al humano.
> 7. Guardar en disco con **Tool** `write_file`.
> 8. (Fin del Workflow)

---

## Ver también

- [[Arquitectura Cognitiva de Agentes]]
- [[Skills (Habilidades) en IA]]
